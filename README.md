# php-xss-attacks

1- Dosya isimleri php uzantılı olmalı, yorumlayıcı sadece php olunca sensitive verileri gizliyor  
[video-1](https://www.youtube.com/watch?v=mCwAsvNdPRs&list=PLfdtiltiRHWFsPxAGO-SVPGhCbCwKWF_N&index=1). 

2-Veritabanından alınan veriler ekrana yazdırılırken mutlaka escape edilmeli, db ye bir şekilde inject edilmiş zararlı yazılım varsa bunların kullanıcıda çalıştırılması engellenmesi için gerekli.  
[video-2](https://www.youtube.com/watch?v=mCwAsvNdPRs&list=PLfdtiltiRHWFsPxAGO-SVPGhCbCwKWF_N&index=2). 

```php
   functions.php
	<?php
		// escape value
		function e($string){
  			return htmlspecialchars($string, ENT_QUOTES, ‘UTF-8’);
		}
	?>
```     
```php
	form.php {ORNEK BİR FORM SAYFASI İÇİN ORNEK KOD}
	<?php
		require ‘functions.php’
	?>
	<?php echo e($inputValue); ?>
	
        NOT: Sayfalarda head alanında <meta charset=“UTF-8”> olmalı
```


3-Şifreler MD5 gibi decrypt edilebilen algoritmala ile db ye kayıt edilmemeli,
Şifreler hashlenip db ye kayıt edilmeli, login işleminde şifre hashlenip db de karşılaştırılmalı.  
[video-3](https://www.youtube.com/watch?v=mCwAsvNdPRs&list=PLfdtiltiRHWFsPxAGO-SVPGhCbCwKWF_N&index=3)  
```php
  functions.php
	<?php
               // password-hash
		function ph($psw){
  			return password_hash($psw, PASSWORD_DEFAULT, [‘cost’ => 12]);
		}
		// password-verify
		function pv($pswDb, $pswSubmited){
  			return password_verify($pswSubmited, $psw);
		}
	?>
```

4-Uygulama directory listening protect işlemi için .htaccess dosyası ana dizine eklenmeli
[video-4](https://www.youtube.com/watch?v=mCwAsvNdPRs&list=PLfdtiltiRHWFsPxAGO-SVPGhCbCwKWF_N&index=4)  

```php
  .htaccess	// root dizinde olmalı
		
		Options -Indexes
    
```

5-Cookie de veriler açık şekilde tutulmamalı, mutlaka encrypt edilmiş olmalı
[video-5](https://www.youtube.com/watch?v=mCwAsvNdPRs&list=PLfdtiltiRHWFsPxAGO-SVPGhCbCwKWF_N&index=5), 
[video-6](https://www.youtube.com/watch?v=mCwAsvNdPRs&list=PLfdtiltiRHWFsPxAGO-SVPGhCbCwKWF_N&index=6).  


6-CSRF atakları için ilk önlem, form actionlarında isteklerin gerçekte olması gereken metodlar ile istekte bulunup bulunmadığı kontrol edilmeli.(POST, DELETE, PUT, GET).  
[video-7](https://www.youtube.com/watch?v=mCwAsvNdPRs&list=PLfdtiltiRHWFsPxAGO-SVPGhCbCwKWF_N&index=7)  
```php
  <?php
		if($_SERVER[‘REQUEST_METHOD’] !== ‘’POST){
			die();
		}
	?>
```    

7-CSRF atakları için ikinci önlem, her sayfa isteğinde bootstrap.php (php bootstrapping) dosyasında sessiona form tokenı oluşturulup eklenmeli. form sayfaları içerisinde hidden alana bu tekken değer eklenmeli. Form action çağrıldığında bootstrap.php dosyasında istek türü post,put,patch,delete olanlar için session içerisindeki form token ile formdan gelen token kontrol edilmeli.  

[video-7](https://www.youtube.com/watch?v=mCwAsvNdPRs&list=PLfdtiltiRHWFsPxAGO-SVPGhCbCwKWF_N&index=7)  
```php
  bootstrap.php.php {ORNEK BİR bootstrap.php  SAYFASI İÇİN ORNEK KOD}
	<?php
		session_start();
		
		if($_SERVER[‘REQUEST_METHOD’] === ‘’POST){
			if(!isset($_POST[‘’token]) || ($_POST[‘’token] != $_SESSION[‘’_token]))
			die(‘’Invalid CSRF token);
		}

		$_SESSION[‘’_token] = bin2hex(openssl_random_pseudo_bytes(16));
	?>
``` 
```php
  herhangi_form.php
	 <form>
		//...
		<input type="hidden" name="token" value="<?php echo $_SESSION['token']; ?>" />
		//...
	</form>
```
