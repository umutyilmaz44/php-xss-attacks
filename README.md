# php-xss-attacks

**_1- PHP Security: Include/require file extensions:_**  
> Dosya isimleri php uzantılı olmalı, yorumlayıcı sadece php olunca sensitive verileri gizliyor  
> <a href="https://www.youtube.com/watch?v=mCwAsvNdPRs&list=PLfdtiltiRHWFsPxAGO-SVPGhCbCwKWF_N&index=1" target="_blank">video-1</a>. 


**_2- PHP Security: XSS (Cross-site Scripting):_**
> Veritabanından alınan veriler ekrana yazdırılırken mutlaka escape edilmeli, db ye bir şekilde inject edilmiş zararlı yazılım varsa bunların kullanıcıda çalıştırılması engellenmesi için gerekli.  
> [video-2](https://www.youtube.com/watch?v=mCwAsvNdPRs&list=PLfdtiltiRHWFsPxAGO-SVPGhCbCwKWF_N&index=2). 

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


**_3- PHP Security: Password hashing:_**
> Şifreler MD5 gibi decrypt edilebilen algoritmala ile db ye kayıt edilmemeli, Şifreler hashlenip db ye kayıt edilmeli, login işleminde şifre hashlenip db de karşılaştırılmalı.  
> [video-3](https://www.youtube.com/watch?v=mCwAsvNdPRs&list=PLfdtiltiRHWFsPxAGO-SVPGhCbCwKWF_N&index=3)  
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

**_4- PHP Security: Directory listing:_** 
> Uygulama directory listening protect işlemi için .htaccess dosyası ana dizine eklenmeli
> [video-4](https://www.youtube.com/watch?v=mCwAsvNdPRs&list=PLfdtiltiRHWFsPxAGO-SVPGhCbCwKWF_N&index=4)  

```php
  .htaccess	// root dizinde olmalı
		
		Options -Indexes
    
```

:**_5- PHP Security: HttpOnly Cookies:_**
> Cookie de veriler açık şekilde tutulmamalı, mutlaka encrypt edilmiş olmalı
> [video-5](https://www.youtube.com/watch?v=mCwAsvNdPRs&list=PLfdtiltiRHWFsPxAGO-SVPGhCbCwKWF_N&index=5), 
> [video-6](https://www.youtube.com/watch?v=mCwAsvNdPRs&list=PLfdtiltiRHWFsPxAGO-SVPGhCbCwKWF_N&index=6).  


**_6- PHP Security: What you shouldn't store in cookies:_**
> CSRF atakları için ilk önlem, form actionlarında isteklerin gerçekte olması gereken metodlar ile istekte bulunup bulunmadığı kontrol edilmeli.(POST, DELETE, PUT, GET).  
> [video-7](https://www.youtube.com/watch?v=mCwAsvNdPRs&list=PLfdtiltiRHWFsPxAGO-SVPGhCbCwKWF_N&index=7)  
```php
  <?php
		if($_SERVER[‘REQUEST_METHOD’] !== ‘’POST){
			die();
		}
	?>
```    

**_7- PHP Security: CSRF (Cross-site Request Forgery):_**
> CSRF atakları için ikinci önlem, her sayfa isteğinde bootstrap.php (php bootstrapping) dosyasında sessiona form tokenı oluşturulup eklenmeli. form sayfaları içerisinde hidden alana bu tekken değer eklenmeli. Form action çağrıldığında bootstrap.php dosyasında istek türü post,put,patch,delete olanlar için session içerisindeki form token ile formdan gelen token kontrol edilmeli.  

>[video-7](https://www.youtube.com/watch?v=mCwAsvNdPRs&list=PLfdtiltiRHWFsPxAGO-SVPGhCbCwKWF_N&index=7)  
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

**_8- PHP Security: User defined file includes:_** 
> Sensitive (hassas, özel) veri içeren php dosyaları file_get_contents gibi metodlarla dinamik olarak okunmamalı.   
> [video-8](https://www.youtube.com/watch?v=mCwAsvNdPRs&list=PLfdtiltiRHWFsPxAGO-SVPGhCbCwKWF_N&index=8) 

```php
  herhangi_dosya.php
	<?php
	if(!isset($_GET['show'])){
	  die();
	}
	$show = strtolower(trim($_GET['show']));
	
	$allowed = ['product','order','delivery'];
	
	$content = in_array($showi $allowed) ? file_get_contents('content/{$show}.php') : '';
	?>
	
	<!DOCTYPE html>
	<html lang="en">
		<head>
		<meta charset="UTF-8">
		<title>Dynamic file content loading</title>
		</head>
		<body>
			<?php echo $content; ?>
		</body>
	</html>
```

**_9- PHP Security: SQL Injection:_**
> form inputlarından gelen veriler ile oluşturulan sql ifadelerinde input verileri doğrudan eklenmemeli
> [video-9](https://www.youtube.com/watch?v=mCwAsvNdPRs&list=PLfdtiltiRHWFsPxAGO-SVPGhCbCwKWF_N&index=9) 
```php
	login.php
	<?php
	$db = new PDO('mysql:host=localhost:127.0.0.1;dbname=website;','root','root_psw');
	if(isset($_GET['email'])){
	  $email=$_GET['email'];
	  
	  // User control code examples are below.
	  
	  if($user->rowCount()){
	  	// login operation success
	  }
	}
	?>
	
	<!DOCTYPE html>
	<html lang="en">
		<head>
		<meta charset="UTF-8">
		<title>Login User</title>
		</head>
		<body>
			<form action="login.php" method="POST" autocomplete="off">
				<label for="email">
					Email
					<input type="text" name="email" id="email">
				</label>
				<input type="submit" value="login">
			</form>
		</body>
	</html>
```

NOT SECURE
```php
	$user = db->query('SELECT * FROM user WHERE email={$email}');
```

SECURE
```php
	 $user = db->prepare('SELECT * FROM user WHERE email = :email');
	 $user->execute([
	 	'email' => mysql_real_escape_string($email),
	 ]);
```

**_10- PHP Security: Error Reporting:_**
> php.ini konfigurasyon dosyasındaki **display_errors** özelliğ **off** olmalı, aksi takdirde sensitive veriler kullanıcı tarafında ele geçirilebilir. Hata durumlarını loglama ile sistem yöneticisi tarafından daha güvenli takibi sağlanabilir
> [video-10](https://www.youtube.com/watch?v=mCwAsvNdPRs&list=PLfdtiltiRHWFsPxAGO-SVPGhCbCwKWF_N&index=10) 
