**Write-up du 3615 Incident (1) from ECSC**

Très bien, tout d'abord, nous avons un dump mémoire à inspecter.

Commençons par récuperer les profiles avec volatility

```
          Suggested Profile(s) : Win10x64_10586, Win10x64_14393, Win10x64, Win2016x64_14393, Win10x64_15063 (Instantiated with Win10x64_15063)
                     AS Layer1 : SkipDuplicatesAMD64PagedMemory (Kernel AS)
                     AS Layer2 : WindowsCrashDumpSpace64 (Unnamed AS)
                     AS Layer3 : FileAddressSpace (/home/l0uky/Desktop/Dossier privé/CTF/ecsc/mem.dmp)
                      PAE type : No PAE
                           DTB : 0x1ab000L
                          KDBG : 0xf801f433ba60L
          Number of Processors : 2
     Image Type (Service Pack) : 0
                KPCR for CPU 0 : 0xfffff801f4394000L
                KPCR for CPU 1 : 0xffffd0012eb07000L
             KUSER_SHARED_DATA : 0xfffff78000000000L
           Image date and time : 2019-05-08 20:04:11 UTC+0000
     Image local date and time : 2019-05-08 22:04:11 +0200
```

Nous avons plusieurs profiles qui s'offrent à nous

On va commencer par essayer d'avoir le nom de l'ordinateur de la victime !

```
l0uky@parrot: ~/Desktop/Dossier privé/CTF/ecsc $ strings mem.dmp | grep "^COMPUTERNAME" | head -n 1 
COMPUTERNAME=DESKTOP-704QVQQ
```

Nous avons le nom de l'ordinateur.

Bien, continuons, inspectons les processus en cours d'execution.

```
l0uky@parrot: ~/Desktop/Dossier privé/CTF/ecsc $ volatility --profile=Win10x64_10586 -f mem.dmp pstree
Volatility Foundation Volatility Framework 2.6
Name                                                  Pid   PPid   Thds   Hnds Time
-------------------------------------------------- ------ ------ ------ ------ ----
 0xffffe0000f65a040:System                              4      0    136      0 2019-05-08 19:57:03 UTC+0000
. 0xffffe00010e4b040:smss.exe                         256      4      3      0 2019-05-08 19:57:03 UTC+0000
 0xffffe00011344080:winlogon.exe                      544    464      5      0 2019-05-08 19:57:05 UTC+0000
. 0xffffe00012034080:userinit.exe                    3120    544      0 ------ 2019-05-08 19:57:14 UTC+0000
.. 0xffffe000116e3080:explorer.exe                   3184   3120     86      0 2019-05-08 19:57:14 UTC+0000
... 0xffffe00012774080:OneDrive.exe                  3080   3184     17      0 2019-05-08 19:57:29 UTC+0000
... 0xffffe00012854840:MSASCui.exe                   5840   3184      6      0 2019-05-08 20:01:01 UTC+0000
... 0xffffe000125a7840:firefox.exe                   4040   3184     59      0 2019-05-08 19:59:06 UTC+0000
.... 0xffffe00010385080:firefox.exe                  4736   4040     20      0 2019-05-08 19:59:08 UTC+0000
.... 0xffffe00011196080:firefox.exe                  3256   4040     22      0 2019-05-08 19:59:11 UTC+0000
.... 0xffffe00010347080:firefox.exe                  3744   4040     19      0 2019-05-08 19:59:09 UTC+0000
.... 0xffffe000125f7840:firefox.exe                  4896   4040      9      0 2019-05-08 19:59:07 UTC+0000
.... 0xffffe00012155200:firefox.exe                  1360   4040     19      0 2019-05-08 19:59:42 UTC+0000
.... 0xffffe000127446c0:firefox.exe                  5084   4040      0 ------ 2019-05-08 19:59:33 UTC+0000
... 0xffffe00012620080:vmtoolsd.exe                  4812   3184     10      0 2019-05-08 19:57:27 UTC+0000
... 0xffffe0001214e080:notepad++.exe                 5496   3184      0 ------ 2019-05-08 20:00:33 UTC+0000
... 0xffffe00012268100:notepad.exe                   5444   3184      1      0 2019-05-08 20:00:29 UTC+0000
... 0xffffe000106bb840:assistance.exe                5208   3184      9      0 2019-05-08 20:00:16 UTC+0000
.... 0xffffe00010335080:conhost.exe                  5224   5208      2      0 2019-05-08 20:00:16 UTC+0000
... 0xffffe0001051c840:DumpIt.exe                    5596   3184      6      0 2019-05-08 20:04:09 UTC+0000
.... 0xffffe0001051b080:conhost.exe                  5364   5596      4      0 2019-05-08 20:04:09 UTC+0000
... 0xffffe0001287a840:notepad++.exe                 5176   3184     11      0 2019-05-08 20:01:49 UTC+0000
. 0xffffe00011739080:dwm.exe                          836    544     13      0 2019-05-08 19:57:06 UTC+0000
 0xffffe00011305180:csrss.exe                         480    464     13      0 2019-05-08 19:57:05 UTC+0000
 0xffffe00012530080:MpCmdRun.exe                     3248   4932      7      0 2019-05-08 19:59:43 UTC+0000
 0xffffe00011302080:wininit.exe                       472    348      4      0 2019-05-08 19:57:05 UTC+0000
. 0xffffe00011399840:services.exe                     592    472     20      0 2019-05-08 19:57:05 UTC+0000
.. 0xffffe0000f685840:svchost.exe                    1036    592     30      0 2019-05-08 19:57:06 UTC+0000
.. 0xffffe0000f823340:msdtc.exe                      2464    592     13      0 2019-05-08 19:57:10 UTC+0000
.. 0xffffe0000f839840:NisSrv.exe                     2708    592     12      0 2019-05-08 19:57:10 UTC+0000
.. 0xffffe00011617840:spoolsv.exe                    1304    592     16      0 2019-05-08 19:57:06 UTC+0000
.. 0xffffe000117e1080:vmacthlp.exe                    668    592      4      0 2019-05-08 19:57:06 UTC+0000
.. 0xffffe00011cf1840:VGAuthService.                 1712    592      3      0 2019-05-08 19:57:07 UTC+0000
.. 0xffffe000117e0840:svchost.exe                     296    592     19      0 2019-05-08 19:57:06 UTC+0000
... 0xffffe000126b7840:WUDFHost.exe                  6100    296     10      0 2019-05-08 20:01:27 UTC+0000
.. 0xffffe000113dd480:svchost.exe                     684    592     28      0 2019-05-08 19:57:05 UTC+0000
... 0xffffe000125b8080:SearchUI.exe                  3888    684     61      0 2019-05-08 20:00:03 UTC+0000
... 0xffffe000125fb840:WmiPrvSE.exe                  4916    684     12      0 2019-05-08 19:57:28 UTC+0000
... 0xffffe00012077240:SkypeHost.exe                 3220    684     37      0 2019-05-08 19:57:14 UTC+0000
... 0xffffe00011f8f7c0:ShellExperienc                3576    684     28      0 2019-05-08 19:57:15 UTC+0000
... 0xffffe000115ae840:WmiPrvSE.exe                  2244    684     10      0 2019-05-08 19:57:09 UTC+0000
... 0xffffe00012023580:RuntimeBroker.                3092    684     23      0 2019-05-08 19:57:14 UTC+0000
.. 0xffffe00011779840:svchost.exe                     944    592     76      0 2019-05-08 19:57:06 UTC+0000
... 0xffffe00010aba840:sihost.exe                    2204    944     14      0 2019-05-08 19:57:14 UTC+0000
... 0xffffe00010441600:taskhostw.exe                 3192    944      7      0 2019-05-08 20:02:15 UTC+0000
... 0xffffe00011fa8840:taskhostw.exe                 2168    944     11      0 2019-05-08 19:57:14 UTC+0000
.. 0xffffe00011cff840:vmtoolsd.exe                   1732    592     10      0 2019-05-08 19:57:07 UTC+0000
.. 0xffffe0001225b840:SearchIndexer.                 3444    592     31      0 2019-05-08 19:57:15 UTC+0000
... 0xffffe00011f8b080:SearchProtocol                5060   3444      7      0 2019-05-08 19:59:31 UTC+0000
... 0xffffe000123e21c0:SearchFilterHo                4320   3444      4      0 2019-05-08 20:02:52 UTC+0000
.. 0xffffe0000f683840:svchost.exe                    1216    592     21      0 2019-05-08 19:57:06 UTC+0000
.. 0xffffe00011d0a840:svchost.exe                    1760    592     11      0 2019-05-08 19:57:07 UTC+0000
.. 0xffffe00011789840:svchost.exe                     964    592     23      0 2019-05-08 19:57:06 UTC+0000
... 0xffffe000126d3080:audiodg.exe                   2624    964      8      0 2019-05-08 20:00:15 UTC+0000
.. 0xffffe000115ac840:dllhost.exe                    2308    592     16      0 2019-05-08 19:57:09 UTC+0000
.. 0xffffe0001178c840:svchost.exe                     972    592     25      0 2019-05-08 19:57:06 UTC+0000
.. 0xffffe000122aa840:svchost.exe                    4452    592      9      0 2019-05-08 19:57:23 UTC+0000
.. 0xffffe000113f2180:svchost.exe                     740    592     13      0 2019-05-08 19:57:06 UTC+0000
.. 0xffffe00011d1b840:MsMpEng.exe                    1776    592     42      0 2019-05-08 19:57:07 UTC+0000
.. 0xffffe0001179c840:svchost.exe                    1000    592      9      0 2019-05-08 19:57:06 UTC+0000
.. 0xffffe00011cc45c0:svchost.exe                    1652    592     13      0 2019-05-08 19:57:07 UTC+0000
.. 0xffffe00012910080:svchost.exe                    5792    592      4      0 2019-05-08 20:00:58 UTC+0000
. 0xffffe000113a2840:lsass.exe                        604    472     10      0 2019-05-08 19:57:05 UTC+0000
 0xffffe00010ef2080:csrss.exe                         360    348     11      0 2019-05-08 19:57:05 UTC+0000
```

Bon, nous avons les PIDs, mais nous n'avons pas de processus suspect. Cherchons quelque chose directement dans le fichier. 

Bon, jusqu'à là, on sait que nous devons trouver le flag dans un .docx, essayons de grep tout les docx

```
l0uky@parrot: ~/Desktop/Dossier privé/CTF/ecsc $ strings mem.dmp | grep "docx"
LibreOffice.Docx_.docx
.docx
docxfile
n.docx
.docx
.docx
docxfile
mes d'information_fichiers\twitter_logo.svg<br>C:\Users\TNKLSAI3TGT7O9\Documents\flag.docx<br>C:\Users\TNKLSAI3TGT7O9\Documents\simple_document.docx<br>C:\Users\TNKLSAI3TGT7O9\Favorites\Links\desktop.ini<br>C:\Users\TNKLSAI3TGT7O9\Links\desktop.ini<br>C:\Users\TNKLSAI3TGT7O9\Favorites\desktop.ini<br>C:\Users\TNKLSAI3TGT7O9\Music\desktop.ini<br>C:\Users\TNKLSAI3TGT7O9\OneDrive\desktop.ini<br>C:\Users\TNKLSAI3TGT7O9\Pictures\Camera Roll\desktop.ini<br>C:\Users\TNKLSAI3TGT7O9\Pictures\Saved Pictures\desktop.ini<br>C:\Users\TNKLSAI3TGT7O9\Pictures\desktop.ini<br>C:\Users\TNKLSAI3TGT7O9\Saved Games\desktop.ini<br>C:\Users\TNKLSAI3TGT7O9\Searches\desktop.ini<br>C:\Users\TNKLSAI3TGT7O9\Videos\desktop.ini<br>C:\Users\TNKLSAI3TGT7O9\ntuser.ini<br>C:\Users\desktop.ini
.doc".docx".docm".xls".xlsx".xlsm".ppt".pptx".pptm
docx
docx
	m=] n=asn1avx2basebindbmi1bmi2boolcallcas1cas2cas3cas4cas5cas6chanconfdatedeaddialdocxepubermsetagfilefromftpsfuncgziphosthourhtmlhttpicmpidleigmpinddint8javajpegjsonkindlinknonenullopenpop3pptxreadsbrkscvgsmtpsse2sse3tag:tcp4tcp6trueudp6uintunixvaryxlsxxn-- ...
docxfile_.docx
cuter","Type":12}},{"System.FileExtension":{"Value":".exe","Type":12},"System.Software.ProductVersion":{"Value":"N/A","Type":12},"System.Kind":{"Value":"program","Type":12},"System.ParsingName":{"Value":"TheDocumentFoundation.LibreOffice.Writer","Type":12},"System.IsFlagged":{"Value":1,"Type":5},"System.Software.TimesUsed":{"Value":0,"Type":5},"System.Tile.Background":{"Value":16777215,"Type":5},"System.Software.AppId":{"Value":"{A76E406A-C225-405C-A323-02D8F1E4E2C6}","Type":16},"System.Link.Arguments":{"Value":"N/A","Type":12},"System.Identity":{"Value":"N/A","Type":12},"System.FileName":{"Value":"swriter","Type":12},"System.ConnectedSearch.JumpList":{"Value":"[{\"Type\":0,\"Items\":[{\"Type\":1,\"Name\":\"flag\",\"Path\":\"C:\\\\Users\\\\TNKLSAI3TGT7O9\\\\Documents\\\\flag.docx\",\"Date\":\"2019-5-8\",\"Points\":3},{\"Type\":1,\"Name\":\"simple_document\",\"Path\":\"C:\\\\Users\\\\TNKLSAI3TGT7O9\\\\Documents\\\\simple_document.docx\",\"Date\":\"2019-5-8\",\"Points\":1}]}]","Type":12},"System.ItemType":{"Value":"Desktop","Type":12},"System.DateAccessed":{"Value":1.3201814954744298E+17,"Type":14},"System.Tile.EncodedTargetPath":{"Value":"{6D809377-6AF0-444B-8957-A3773F02200E}\\LibreOffice\\program\\swriter.exe","Type":12},"System.Tile.SmallLogoPath":{"Value":"N/A","Type":12},"System.ItemNameDisplay":{"Value":"LibreOffice Writer","Type":12}},{"System.FileExtension":{"Value":"N/A","Type":12},"System.Software.ProductVersion":{"Value":"10.1510.9010.0","Type":12},"System.Kind":{"Value":"program","Type":12},"System.ParsingName":{"Value":"Microsoft.WindowsPhone_8wekyb3d8bbwe!CompanionApp.App","Type":12},"System.IsFlagged":{"Value":1,"Type":5},"System.Software.TimesUsed":{"Value":0,"Type":5},"System.Tile.Background":{"Value":16777215,"Type":5},"System.Software.AppId":{"Value":"{A84185CD-B4FF-4C79-987E-F8958B9177B5}","Type":16},"System.Link.Arguments":{"Value":"N/A","Type":12},"System.Identity":{"Value":"Microsoft.WindowsPhone_8wekyb3d8bbwe","Type":12},"System.FileName":{"Value":"N/A","Type":12},"System.ConnectedSearch.JumpList":{"Value":"[]","Type":12},"System.ItemType":{"Value":"Trusted Immersive","Type":12},"System.DateAccessed":{"Value":1.3201814540629707E+17,"Type":14},"System.Tile.EncodedTargetPath":{"Value":"N/A","Type":12},"System.Tile.SmallLogoPath":{"Value":"@{Microsoft.WindowsPhone_10.1510.9010.0_x64__8wekyb3d8bbwe?ms-resource://Microsoft.WindowsPhone/Files/Assets/CompanionAppAppList.png}","Type":12},"System.ItemNameDisplay":{"Value":"Assistant Mobile","Type":12}},{"System.FileExtension":{"Value":"N/A","Type":12},"System.Software.ProductVersion":{"Value":"10.0.2840.0","Type":12},"System.Kind":{"Value":"program","Type":12},"System.ParsingName":{"Value":"Microsoft.People_8wekyb3d8bbwe!x4c7a3b7dy2188y46d4ya362y19ac5a5805e5x","Type":12},"System.IsFlagged":{"Value":1,"Type":5},"System.Software.TimesUsed":{"Value":0,"Type":5},"System.Tile.Background":{"Value":16777215,"Type":5},"System.Software.AppId":{"Value":"{B07892FA-4240-49F1-958A-67E8588FF2CE}","Type":16},"System.Link.Arguments":{"Value":"N/A","Type":12},"System.Identity":{"Value":"Microsoft.People_8wekyb3d8bbwe","Type":12},"System.FileName":{"Value":"N/A","Type":12},"System.ConnectedSearch.JumpList":{"Value":"[]","Type":12},"System.ItemType":{"Value":"Trusted Immersive","Type":12},"S
C:\Users\TNKLSAI3TGT7O9\Documents\simple_document.docx
Walking C:\Users\TNKLSAI3TGT7O9\Music\desktop.ini.docx...
C:\Users\TNKLSAI3TGT7O9\Documents\simple_document.docx
C:\Users\TNKLSAI3TGT7O9\Documents\simple_document.docx
docx
docx
C:\Users\TNKLSAI3TGT7O9\Documents\flag.docx
C:\Users\TNKLSA~1\AppData\Local\Temp\flag.docxniWalking C:\Users\TNKLSAI3TGT7O9\Favorites
C:\Users\TNKLSA~1\AppData\Local\Temp\flag.docxg
C:\Users\TNKLSAI3TGT7O9\Documents\flag.docx
dmlzdWVsLWFybmFxdWUtbm90cmUtZGFtZS5qcGc=g.docx
C:\Users\TNKLSAI3TGT7O9\Documents\flag.docx
C:\Users\TNKLSAI3TGT7O9\Documents\flag.docx
.docx
```

Nous voyons que nous avons un fichier "flag.docx", nous avons donc le fichier où se trouve le flag.

Essayons d'en chercher plus...

```
l0uky@parrot: ~/Desktop/Dossier privé/CTF/ecsc $ strings mem.dmp | grep "flag.docx" -C 25
LOGONSERVER=\\DESKTOP-704QVQQ
OneDrive=C:\Users\TNKLSAI3TGT7O9\OneDrive
ProgramFiles=C:\Program Files (x86)
ProgramFiles(x86)=C:\Program Files (x86)
ProgramW6432=C:\Program Files
TEMP=C:\Users\TNKLSA~1\AppData\Local\Temp
TMP=C:\Users\TNKLSA~1\AppData\Local\Temp
USERDOMAIN_ROAMINGPROFILE=DESKTOP-704QVQQ
USERPROFILE=C:\Users\TNKLSAI3TGT7O9
dmlzdWVsLWFybmFxdWUtbm90cmUtZGFtZS5qcGc=
dmlzdWVsLWFybmFxdWUtbm90cmUtZGFtZS5qcGc=g.docx
icon_administrationBlanc_mini.png
aWNvbl9hZG1pbmlzdHJhdGlvbkJsYW5jX21pbmkucG5n
aWNvbl9hZG1pbmlzdHJhdGlvbkJsYW5jX21pbmkucG5njs
aWNvbl9lbnRyZXByaXNlQmxhbmNfbWluaS5wbmc=
aWNvbl9lbnRyZXByaXNlQmxhbmNfbWluaS5wbmc=p.ini
aWNvbl9wYXJ0aWN1bGllckJsYW5jX21pbmkucG5n
aWNvbl9wYXJ0aWN1bGllckJsYW5jX21pbmkucG5np.ini
C:\Users\TNKLSAI3TGT7O9\Documents\ZmxhZy5kb2N4
C:\Users\TNKLSAI3TGT7O9\Documents\ZmxhZy5kb2N4
C:\Users\TNKLSAI3TGT7O9\Documents\flag.docx
C:\Users\TNKLSAI3TGT7O9\Documents\flag.docx
.iniC:\Users\TNKLSAI3TGT7O9\Links\ZGVza3RvcC5pbmk=
C:\Users\TNKLSAI3TGT7O9\Links\ZGVza3RvcC5pbmk=jsC:\Users\TNKLSAI3TGT7O9\Links\desktop.ini
op.iniC:\Users\TNKLSAI3TGT7O9\Links\desktop.ini
C:\Users\TNKLSAI3TGT7O9\Favorites\desktop.ini
niC:\Users\TNKLSAI3TGT7O9\Favorites\desktop.ini
C:\Users\TNKLSAI3TGT7O9\Music\ZGVza3RvcC5pbmk=
C:\Users\TNKLSAI3TGT7O9\Music\ZGVza3RvcC5pbmk=
C:\Users\TNKLSAI3TGT7O9\Music\desktop.ini
op.iniC:\Users\TNKLSAI3TGT7O9\Music\desktop.ini
C:\Users\TNKLSAI3TGT7O9\OneDrive\desktop.ini
iniC:\Users\TNKLSAI3TGT7O9\OneDrive\desktop.ini
iniC:\Users\TNKLSAI3TGT7O9\Pictures\desktop.ini
C:\Users\TNKLSAI3TGT7O9\Pictures\desktop.ini
C:\Users\TNKLSAI3TGT7O9\Pictures\desktop.ini
C:\Users\TNKLSAI3TGT7O9\Saved Games\desktop.ini
C:\Users\TNKLSAI3TGT7O9\Saved Games\desktop.ini
C:\Users\TNKLSAI3TGT7O9\Searches\desktop.ini
C:\Users\TNKLSAI3TGT7O9\Searches\desktop.ini
C:\Users\TNKLSAI3TGT7O9\Videos\ZGVza3RvcC5pbmk=
C:\Users\TNKLSAI3TGT7O9\Videos\ZGVza3RvcC5pbmk=
```

Nous avons plusieurs fichier, dont des fichiers avec un nom en base64; mon esprit curieux m'a torturé, c'est donc pour cela qu'on va decider les b64.

```
l0uky@parrot: ~ $ echo "dmlzdWVsLWFybmFxdWUtbm90cmUtZGFtZS5qcGc=" | base64 -d
visuel-arnaque-notre-dame.jpg

l0uky@parrot: ~/Desktop/Dossier privé/CTF/ecsc $ echo "ZGVza3RvcC5pbmk=" | base64 -d
desktop.ini

l0uky@parrot: ~/Desktop/Dossier privé/CTF/ecsc $ echo "ZmxhZy5kb2N4" | base64 -d 
flag.docx
```

Nous avons donc trois hash de fichiers, que pouvons nous faire avec?

Tentons de chercher le hash du flag.docx dans notre mem.dmp

```
l0uky@parrot: ~/Desktop/Dossier privé/CTF/ecsc $ strings mem.dmp | grep "ZmxhZy5kb2N4" 
ZmxhZy5kb2N4.
C:\Users\TNKLSAI3TGT7O9\Downloads\ZmxhZy5kb2N4.chiffr
C:\Users\TNKLSAI3TGT7O9\Documents\ZmxhZy5kb2N4.chiffr
ZmxhZy5kb2N4.chiffr
ZmxhZy5kb2N4.chiffr
ZmxhZy5kb2N4.lnk
ZmxhZy5kb2N4.lnk
```
Un fichier a attiré mon attention "ZmxhZy5kb2N4.chiffr"

L'output n'est pas en utf-8 donc le "é" n'est pas printable.

En inspectant un peu avec volatility, je me suis rendu compte qu'il faisait des recherches sur internet avant que l'attaque soit lancée, et le programme qui s'est lancé s'appellait "assistance.exe"

J'ai chercher le PID, et j'ai trouvé ! 

Le flag était donc ECSC{assistance.exe:5208:c9a12b109a58361ff1381fceccdcdcade3ec595a}
