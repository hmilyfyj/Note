---
title: Docker 空间占用清理
date:  2017-07-10 12:10:19
tags: Note
categories: Note
grammar_cjkRuby: true
---

## 占用
执行 ` df -h 和 du -h --max-depth=1 ` 汇总后得到资源主要由以下内容占用：
>
1.devicemapper
2.logs
3.容器内日志
4.无用的 volumes

## 解决措施

### 针对 devicemapper

### 针对 容器日志
````
#!/bin/bash

for container_id in $(docker ps -a --filter="name=$name" -q);

    do file=$(docker inspect $container_id | grep -G '"LogPath": "*"' | sed -e 's/.*"LogPath": "//g' | sed -e 's/",//g');

    if [ -f $file ]
      then
	 echo $file;
    fi

done

````

### 针对 容器内日志

### 无用的 volumes
````bash
#! /bin/bash

set -eou pipefail

#usage: sudo ./docker-cleanup-volumes.sh [--dry-run]

docker_bin="$(which docker.io 2> /dev/null || which docker 2> /dev/null)"

# Default dir
dockerdir=/var/lib/docker

# Look for an alternate docker directory with -g/--graph option
dockerpid=$(ps ax | grep "$docker_bin" | grep -v grep | awk '{print $1; exit}') || :
if [[ -n "$dockerpid" && $dockerpid -gt 0 ]]; then
    next_arg_is_dockerdir=false
    while read -d $'\0' arg
    do
        if [[ $arg =~ ^--graph=(.+) ]]; then
            dockerdir=${BASH_REMATCH[1]}
            break
        elif [ $arg = '-g' ]; then
            next_arg_is_dockerdir=true
        elif [ $next_arg_is_dockerdir = true ]; then
            dockerdir=$arg
            break
        fi
    done < /proc/$dockerpid/cmdline
fi

dockerdir=$(readlink -f "$dockerdir")

volumesdir=${dockerdir}/volumes
vfsdir=${dockerdir}/vfs/dir
allvolumes=()
dryrun=false
verbose=false

function log_verbose() {
    if [ "${verbose}" = true ]; then
        echo "$1"
    fi;
}

function delete_volumes() {
  local targetdir=$1
  echo
  if [[ ! -d "${targetdir}" || ! "$(ls -A "${targetdir}")" ]]; then
        echo "Directory ${targetdir} does not exist or is empty, skipping."
        return
  fi
  echo "Delete unused volume directories from $targetdir"
  local dir
  while read -d $'\0' dir
  do
        dir=$(basename "$dir")
        if [[ -d "${targetdir}/${dir}/_data" || "${dir}" =~ [0-9a-f]{64} ]]; then
                if [ ${#allvolumes[@]} -gt 0 ] && [[ ${allvolumes[@]} =~ "${dir}" ]]; then
                        echo "In use ${dir}"
                else
                        if [ "${dryrun}" = false ]; then
                                echo "Deleting ${dir}"
                                rm -rf "${targetdir}/${dir}"
                        else
                                echo "Would have deleted ${dir}"
                        fi
                fi
        else
                echo "Not a volume ${dir}"
        fi
  done < <(find "${targetdir}" -mindepth 1 -maxdepth 1 -type d -print0 2>/dev/null)
}

if [ $UID != 0 ]; then
    echo "You need to be root to use this script."
    exit 1
fi

if [ -z "$docker_bin" ] ; then
    echo "Please install docker. You can install docker by running \"wget -qO- https://get.docker.io/ | sh\"."
    exit 1
fi

while [[ $# > 0 ]]
do
    key="$1"

    case $key in
        -n|--dry-run)
            dryrun=true
        ;;
        -v|--verbose)
            verbose=true
        ;;
        *)
            echo "Cleanup docker volumes: remove unused volumes."
            echo "Usage: ${0##*/} [--dry-run] [--verbose]"
            echo "   -n, --dry-run: dry run: display what would get removed."
            echo "   -v, --verbose: verbose output."
            exit 1
        ;;
    esac
    shift
done

# Make sure that we can talk to docker daemon. If we cannot, we fail here.
${docker_bin} version >/dev/null

container_ids=$(${docker_bin} ps -a -q --no-trunc)

#All volumes from all containers
SAVEIFS=$IFS
IFS=$(echo -en "\n\b")
for container in $container_ids; do
        #add container id to list of volumes, don't think these
        #ever exists in the volumesdir but just to be safe
        allvolumes+=${container}
        #add all volumes from this container to the list of volumes
        log_verbose "Inspecting container ${container}"
        for volpath in $(
                ${docker_bin} inspect --format='{{range $key, $val := .}}{{if eq $key "Volumes"}}{{range $vol, $path := .}}{{$path}}{{"\n"}}{{end}}{{end}}{{if eq $key "Mounts"}}{{range $mount := $val}}{{$mount.Source}}{{"\n"}}{{end}}{{end}}{{end}}' ${container} \
        ); do
                log_verbose "Processing volumepath ${volpath}"
                #try to get volume id from the volume path
                vid=$(echo "${volpath}" | sed 's|.*/\(.*\)/_data$|\1|;s|.*/\([0-9a-f]\{64\}\)$|\1|')
                # check for either a 64 character vid or then end of a volumepath containing _data:
                if [[ "${vid}" =~ ^[0-9a-f]{64}$ || (${volpath} =~ .*/_data$ && ! "${vid}" =~ "/") ]]; then
                        log_verbose "Found volume ${vid}"
                        allvolumes+=("${vid}")
                else
                        #check if it's a bindmount, these have a config.json file in the ${volumesdir} but no files in ${vfsdir} (docker 1.6.2 and below)
                        for bmv in $(find "${volumesdir}" -name config.json -print | xargs grep -l "\"IsBindMount\":true" | xargs grep -l "\"Path\":\"${volpath}\""); do
                                bmv="$(basename "$(dirname "${bmv}")")"
                                log_verbose "Found bindmount ${bmv}"
                                allvolumes+=("${bmv}")
                                #there should be only one config for the bindmount, delete any duplicate for the same bindmount.
                                break
                        done
                fi
        done
done
IFS=$SAVEIFS

delete_volumes "${volumesdir}"
delete_volumes "${vfsdir}"
````