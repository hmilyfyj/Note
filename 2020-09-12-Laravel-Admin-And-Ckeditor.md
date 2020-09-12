---
title: Larvel Admin 接入 CKEditor 4
date: 2020-09-12 11:50:08
tags: Git
categories: Git
---

构造扩展：
```php
<?php

namespace App\Admin\Extensions\Form;

use Encore\Admin\Form\Field;

class CKEditor extends Field
{
    public static $js = [
        '//static.tbxzs.com/libs/ckeditor/4.14.1/ckeditor.js',
        '//static.tbxzs.com/libs/ckeditor/4.14.1/adapters/jquery.js',
    ];

    protected $view = 'admin.ckeditor';

    public function render()
    {
        $this->script = <<<EOF

var options = {
        extraPlugins : 'uploadimage,image2',
        filebrowserImageUploadUrl: '/admin/tools/ckeditor/upload?type=Images&by=drop_or_clipboard_up',
        filebrowserUploadUrl: '/admin/tools/ckeditor/upload?type=Files&by=drop_or_clipboard_up'
        };
        $('textarea.{$this->getElementClassString()}').ckeditor(options);
EOF;

        return parent::render();
    }
}
```

接收上传请求：
```php
   /**
     * 支援 ckeditor 上传控件
     *
     * @author Yingjie Feng <fengit@shanjing-inc.com>
     */
    public function uploadImageForCk(Request $request)
    {
        if (true || config('ckeditor.usingLocalPackageUploadServer', false)) {

            if ($request->hasFile('upload')) {
                //
                $file = $request->file('upload');
                $funcNum = $request->input('CKEditorFuncNum', 0);
                $by = $request->input('by', 'btn_up');  // btn_up 通过按钮上传 drop_up 浏览器拖曳上传
                $data = $request->all();
                $rules = [
                    'upload'    => 'mimes:jpeg,png,gif|max:5120',
                ];
                $messages = [
                    'upload.required' => '必须传入文件',
                    'upload.mimes'    => '文件类型不允许,请上传常规的图片(jpg、png、gif)文件',
                    'upload.max'      => '文件过大,文件大小不得超出5MB',
                ];
                $validator = Validator::make($data, $rules, $messages);
                if ($validator->passes()) {
                    if ($file->isValid()) {
                        $directory       = '/upload/default/' . date("Y") . "/" . date('m');
                        $fileName        = time() . mt_rand(1000, 9999) . "." . $file->getExtension();
                        $storateFullPath = $directory . '/' . $fileName;
                        Storage::disk('oss')->put($storateFullPath, file_get_contents($file->getRealPath()));

                        $_url = Storage::disk('oss')->url($storateFullPath);

                        $_url = str_replace('huigou-img.oss-cn-hangzhou.aliyuncs.com', 'img.tbxzs.com', $_url);
                        $_url = str_replace('http:', 'https:', $_url);
                        if ($by === 'btn_up') {  // 传统表单上传
                            return <<<EOT
<script type="text/javascript">window.parent.CKEDITOR.tools.callFunction({$funcNum}, '{$_url}', 'success!');</script>";
EOT;
                        } elseif ($by === 'drop_or_clipboard_up') {  // 拖曳上传或剪切板上传
                            return response()->json([
                                'uploaded' => 1,
                                'fileName' => $fileName,
                                'url' => $_url,
                            ]);
                        } else {
                            return <<<EOT
<script type="text/javascript">alert('upload with wrong way and config!')</script>
EOT;
                        }
                    }
                }
                $err = $validator->messages()->first();
                $err = $err ? ': '.$err : '';
            }
            return <<<EOT
<script type="text/javascript">alert('upload failed {$err} !')</script>
EOT;
        } else {
            return <<<EOT
<script type="text/javascript">alert('upload not allowed!')</script>
EOT;
        }
    }
```