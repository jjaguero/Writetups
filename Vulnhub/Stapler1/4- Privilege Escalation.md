# Enumeración básica sistema
```css
www-data@red:/tmp$ uname -a
Linux red.initech 4.4.0-21-generic #37-Ubuntu SMP Mon Apr 18 18:34:49 UTC 2016 i686 i686 i686 GNU/Linux
```
# Permisos sudo
```css

```
# Permisos SUID
```css

```
# Tareas cron
```css

```


Antes de todo ya accedimos a la maquina como www-data pero hay que hacer tratamiento de tty

realmente la escalada de privilegios la queria hacer sencilla con un exploit sabiendo que la version de Linux es la 4.4.0-21 busque un exploit de escalada de privilegios

descargue un script donde detecta que exploit se pueden usar y fui probandolos


```css
wget --no-check-certificate https://raw.githubusercontent.com/mzet-/linux-exploit-suggester/master/linux-exploit-suggester.sh -O les.sh

chmod +x les.sh
```


![](obsidian/03_Cybersecurity/Writetups/Vulnhub/Stapler1/images/Pasted%20image%2020260905203309.png)

este exploit lo instale y lo ejecute

![](obsidian/03_Cybersecurity/Writetups/Vulnhub/Stapler1/images/Pasted%20image%2020260905203403.png)

una vez en usuario root busque la flag:

![](obsidian/03_Cybersecurity/Writetups/Vulnhub/Stapler1/images/Pasted%20image%2020260905203429.png)


```
b6b545dc11b7a270f4bad23432190c75162c4a2b
```