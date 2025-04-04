
# Grub enable boot menu

Ieri am inceput sa debug-ez un cluster de Kubernetes care dupa upgrade avea probleme cu kube-proxy,
dadea niste erori legate de iptables si pod-ul se restarta la infinit. 
Am ajuns la concluzia ca ar trebui sa fac downgrade de kernel, asta fiind pe niste VM-uri locale care ruleaza  in Proxmox. 

Din fericire, Proxmox are o facilitate draguta in interfata web de access la consola VM-urilor. 
Din nefericire, multe Linux-uri moderne nu mai afiseaza aceel meniu la boot in care poti alege sa bootezi cu un kernel mai vechi. 
Ma rog, poti apasa nu stiu ce tasta fix in secunda cand booteaza si apare meniul ala, 
o tampenie care castiga cateva secunde la boot. 
Asa ca m-am hotarat sa re-activez meniul de boot din care sa pot alage sa bootez cu un kernel mai vechi.

Ca sa imi reamintesc cum se face asta, am cautat pe internet si am gasit zeci de pagini mentionand mentionand urmatorul proces:

1. se editeaza `/etc/default/grub`
2. se modifica `GRUB_TIMEOUT=0` in `GRUB_TIMEOUT=10` sau alt numar, este in secunde
3. putine pagini mentioneaza sa adaugati `GRUB_TIMEOUT_STYLE=menu`, evident trebuie si asta pentru ca default este `GRUB_TIMEOUT_STYLE=hidden`!
3. se ruleaza `update-grub` si apoi `reboot` si urmarim consola

Si *NIMIC!* Abootat fix la fel ca inainte, fara meniu de boot! Wtf?

In caz ca sunteti curiosi, output-ul comenzii `update-grub` a fost:

```
update-grub
Sourcing file `/etc/default/grub'
Sourcing file `/etc/default/grub.d/50-cloudimg-settings.cfg'
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-6.8.0-57-generic
Found initrd image: /boot/initrd.img-6.8.0-57-generic
Found linux image: /boot/vmlinuz-6.8.0-52-generic
Found initrd image: /boot/initrd.img-6.8.0-52-generic
Warning: os-prober will not be executed to detect other bootable partitions.
Systems on them will not be added to the GRUB boot configuration.
Check GRUB_DISABLE_OS_PROBER documentation entry.
Adding boot menu entry for UEFI Firmware Settings ...
done
```

Ati observat de ce *NIMIC* la boot?

-----

Pentru ca exista fisierul `/etc/default/grub.d/50-cloudimg-settings.cfg` care, 
fiind inclus (`Sourcing file...`) dupa, 
suprascrie setarile definite in `/etc/default/grub`! Asa ca acesta trebuie modificat in acest caz 
si evident apoi totul merge perfect.

Ce m-a surprins este ca absolut nimeni in articolele de pe net nu mentioneaza asta. Morala: 
skill #1 pentru devops/sysadmin: intotdeauna cititi si intelegeti ce scrie pe ecran. 


