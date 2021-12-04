# Deserialize

## (De)serialize 101

What is (de)serialization ?

(De)serialization allows for object portability

serialization: It is the process of translating data structure or object state into byte format → store on disk, dbs, or trasmitted over the network

deserialization: extract data structure from bytes
Object → Serialize → Byte Stream

Byte Stream → Unserialize → Object 

PHP exampple:

- serialize() an object to string
- write string to a file
- unserialize() the file's content back  into an object

Cover a range of languages: Object-oriented programming support serialization

- Python: Pickling/unpickling
- PHP:serializing/unserializing
- Java: serializing/deserializing
- Ruby: marshalling/unmarshalling
- etc

## It's a feature, not a bug!!

Deserialization across different languages will attempt to turn whatever byte stream is provided back into an object

Depending on the object, this can result: 

- Privilege escalation through object properties
- Arbitrary code excecution

Risk: 

An untrusted deserialization user inputs by sending malicious data to be de-serialized → lead to logic manipulation or arbitrary code execution 

## PHP

overview

Introduced via unserialize()

PHP call "magic method" when deserializing : __destruct(), __wakeup()

Magic methods used to form POP chains, like ROP in memory corruption

→ POI

Magic Method 

- All function names starting with __ as "magical" → MUST be declared as public
- Very usefull in OOP
- automatically called when certain conditions are met.

Magic method use with serialization: 

__sleep: called when an object → serialied → must be return to array.

Magic method use with deserialization:

__wakeup: called when an obj is deserialized

_destruct: called when PHP script end and obj is destroyed

__toString: uses obj as string but also can be used to read file or more than that based on function call inside it

```php
<?php

	class MagicMethod{
		public $str = "Hello world";
		public function PrintString()
		{
			echo $this->str."\n";
		}
		public function __construct(){
			echo "__contruct method calling \n";
		}
		public function __destruct(){
			echo "__destruct method calling \n";
		}
		public function __sleep(){
			echo "__sleep method calling \n";
			return array("str");
		}
		public function __wakeup(){
			echo "__wakeup method calling \n";
		}
	}

	$obj = new MagicMethod();
	$obj->PrintString();
	$ser = serialize($obj)."\n";
	echo $ser;
	$unser = unserialize($ser);
	$unser->PrintString();

?>
```

Result

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/aad9ef7a-164c-4878-b318-95b1f96b42ca/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/aad9ef7a-164c-4878-b318-95b1f96b42ca/Untitled.png)

```bash
root@blu3:~/laggg/php# php magic.php 
__contruct method calling 
Hello world
__sleep method calling 
O:11:"MagicMethod":1:{s:3:"str";s:11:"Hello world";}
__wakeup method calling 
Hello world
__destruct method calling 
__destruct method calling 
root@blu3:~/laggg/php#
```

Explain: code flow

- First, call __construct() to construct obj
- Second, call __sleep() when starting serialization
- __wakeup() when execute deserialization and also execute __destruct()

note: 

Mitigations 

Never use unserialize() on anything that can be controlled by a user

Use JSON method to encode/decode data: json_encode(), json_decode()

PHP - Wild 

CVE-2015-8562: Joomla Remote Code Excecution

CVE-2015-7808: vBulletin 5 Unserialize Code Execution

CVE-2015-2171: Slim Framework POI

## Java

Introduced via: ObjectInputStream.readObject()

supply malicious object start POP chain from that object's readObject() method