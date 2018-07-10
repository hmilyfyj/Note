---
title: 2018-7-10-Generate-RSA-Key-In-PHP 
date: 2018-07-10 16:16:37
tags: Nginx
categories: Nginx
---

通过 PHP 生成 RSA 秘钥对。


<!-- more -->

---

```
/**
 *
 *
 * @author Feng Yingjie <fengit@shanjing-inc.com>
 */

function toolsGenerateRsaKey($bits = 2048) {
	function convertKey2Str($key) {
		$keys          = explode("\n", $key);

		foreach ($keys as $k => $v) {
			if (empty($v) || stripos($v, '--') !== false) {
				unset($keys[$k]);
			}
		}

		return implode("", $keys);
	}

	$config = [
		"private_key_bits" => $bits,//位数
		"private_key_type" => OPENSSL_KEYTYPE_RSA,
	];
	$res    = openssl_pkey_new($config);

	//
	if ($res === false) {
		$config['config'] = 'D:\xampp\apache\conf\openssl.cnf';
		$res              = openssl_pkey_new($config);
	}

	openssl_pkey_export($res, $privKey, null, $config);//私钥
	$pubKey = openssl_pkey_get_details($res);
	$pubKey = $pubKey["key"];//公钥
	$arr    = ['privKey' => $privKey, 'pubKey' => $pubKey, 'privKeyStr' => convertKey2Str($privKey), 'pubKeyStr' => convertKey2Str($pubKey)];
	$str    = var_export($arr, true);

	return $arr;
}
```