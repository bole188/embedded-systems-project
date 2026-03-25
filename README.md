# Ovaj projekat opisuje korake potrebne za integraciju modula na _Terasic DE1-SoC_ ploču
# Zadatak
Potrebno je integrisati _Air quality 3 Click_ modul na _Terasic DE1-SoC_ ploču. Navedeni modul (ukoliko je ispravno konfigurisan) se koristi za mjerenje kvaliteta vazduha. Koristiti dostupan drajver te napisati minimalnu _user-space_ aplikaciju za očitavanje vrijednosti sa senzora.  
# Opis dostupnih fajlova  
Priloženi fajlovi u ovom repozitorijumu su neophodni za ispravno generisanje krajnjeg _.img_ fajla.  
## socfpga.rbf
Ovaj fajl sadrži konfiguraciju FPGA dijela SoC-a u binarnom formatu (Raw Binary File). Učitava se pri boot procesu kako bi se definisala logika u FPGA fabrici. Bez njega FPGA dio sistema ostaje neinicijalizovan.  
## boot-env.txt
Tekstualni fajl koji definiše U-Boot okruženje (_environment_ varijable). Sadrži parametre poput boot komandi, kernel argumenata i putanja do fajlova. Koristi se za kontrolu načina na koji sistem pokreće kernel.  
## de1_soc_defconfig
Konfiguracioni fajl za Linux kernel koji definiše sve potrebne opcije za izgradnju kernela za DE1-SoC ploču. U njemu se određuju drajveri, paketi, kernel konfiguracija i sl. Predstavlja početnu tačku za generisanje cijelog embedded Linux sistema.  
## genimage.cfg
Konfiguracija za alat _genimage_ koji kreira finalnu sliku diska (npr. SD kartice). Definiše particije, njihove veličine i koje fajlove treba uključiti u svaku particiju. Koristi se nakon build procesa za pripremu bootabilnog medija.  
## socfpga_cyclone5_de1_soc.dts
Device Tree Source fajl opisuje hardver sistema (CPU, memorija, periferije). Kernel koristi ovaj opis da bi znao kako da komunicira sa hardverom. Specifičan je za Cyclone V SoC na DE1-SoC ploči.  
## de1-soc-handoff.patch
_Patch_ fajl mijenja postojeći izvorni kod (najčešće U-Boot ili kernel) kako bi odgovarao specifičnostima _DE1-SoC_ platforme. Obično uključuje _handoff_ informacije između FPGA i HPS dijela sistema. Primjenjuje se tokom build procesa automatski.  
# Rješenje
Za generisanje svih potrebnih artefakata se koristi alat _Buildroot_.  
_Buildroot_ je alat koji automatski generiše kompletan ugrađeni Linux sistem (_toolchain_, kernel, _root filesystem_) iz izvornog koda.  
Na prvom mjestu potrebno je klonirati _Buildroot_ repozitorijum, korištena verzija za ovaj projekat jeste 2024.02. Potrebno se prebaciti na odgovarajuću granu.  
<pre>git clone https://gitlab.com/buildroot.org/buildroot.git
cd buildroot
git checkout 2024.02  </pre>  
Prethodno opisan _de1-soc-handoff.patch_ fajl se primjenjuje na sistem unošenjem njegove putanje u sledeću konfiguracionu stavku:
<pre>
  Bootloaders  --->
         U-Boot  --->
           Custom U-Boot patches        
</pre>

Prilikom konfiguracije Linux kernela potrebno je uključiti i odgovarajući drajver:
<pre>
  Device Drivers --->
    Industrial I/O support --->
      Chemical Sensors --->
        AMS CCS811 VOC sensor & AMS iAQ-Core VOC sensors
</pre>
