This script will filter logs on given string. using php, python and shell script. 
1) get_log_data.sh
This shell script will download data from server to local. And perform grep on local insteed of Server. 
This will prevent server load of IO operation. 

 rsync -arvz gemadmin@100.114.1.82:/var/www/html/application/logs/ /home/gemadmin/bidlog/82/
 rsync -arvz gemadmin@100.114.1.96:/var/www/html/application/logs/ /home/gemadmin/bidlog/96/
echo "" > $1.log
grep "validation"  /home/gemadmin/bidlog/82/* | grep $1 >> $1.log
grep "validation"  /home/gemadmin/bidlog/96/* | grep $1 >> $1.log

#rsync /home/gemadmin/bidlog -arvz gemadmin@100.70.177.143:/home/gemadmin/
#call php script and read line by line replace encryption.
python read.py $1


2) read.py
This script will call php script on every log lines and descept AES. 

import subprocess,os, sys;
with open(sys.argv[1]+".log") as f:
  content = f.readlines();
  fp =   open(sys.argv[1]+"_filter_.log", "w" );
  for line in content:
    line.split(",");
    if(len(line.split(",")) >= 2):

      fp.write(line.split(",")[0]  + line.split(",")[1] + subprocess.check_output(["php aes.php " + line.split(",")[1] , line.split(",")[1] ],  shell=True) + "\n"  )

  fp.close();


3) aes.php

#!/usr/bin/php
<?php
 $value = $argv[1]; //"fdkfjk";
//var_dump($value);
global $skey;
$skey = "a291694ae2dbb522381564b89483984";

decode($value);
   function encode($value) {
        $text = $value;
        $iv_size = mcrypt_get_iv_size(MCRYPT_RIJNDAEL_256, MCRYPT_MODE_ECB);
        $iv = mcrypt_create_iv($iv_size, MCRYPT_RAND);
        $crypttext = mcrypt_encrypt(MCRYPT_RIJNDAEL_256, $skey, $text, MCRYPT_MODE_ECB, $iv);
        echo trim(safe_b64encode($crypttext));
    }
    function decode($value) {
        $crypttext = safe_b64decode($value);
        $iv_size = mcrypt_get_iv_size(MCRYPT_RIJNDAEL_256, MCRYPT_MODE_ECB);
        $iv = mcrypt_create_iv($iv_size, MCRYPT_RAND);
        $decrypttext = mcrypt_decrypt(MCRYPT_RIJNDAEL_256, "a291694ae2dbb522381564b8" , $crypttext, MCRYPT_MODE_ECB, $iv);
        echo trim($decrypttext);

    }
    function safe_b64encode($string) {

        $data = base64_encode($string);
        $data = str_replace(array('+','/','='),array('-','_',''),$data);
        return $data;
    }
 function safe_b64decode($string) {
        $data = str_replace(array('-','_'),array('+','/'),$string);
        $mod4 = strlen($data) % 4;
        if ($mod4) {
            $data .= substr('====', $mod4);
        }
        return base64_decode($data);
    }

?>


