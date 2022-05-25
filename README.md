# MultiHomed
Konfigurasi Server yang memiliki beberapa interface active ke Internet, dimana masing-masing interface harus aktif untuk melayani koneksi masuk dan keluar pada interface yang sama.

Kemudian juga perlu script untuk kemampuan pindah GW untuk koneksi dalam keluar

Pertama test Ping via jaringan utama
                Jika gagal, maka test Ping via jaringnan cadangan
                                Jika gagal, maka tidak ada keputusan
                                Jika berhasil, maka ganti GW
                Jika berhasil, maka periksa apakah sekarang aktif dijaringan cadangan
                                Jika ya, maka ganti Kembali ke jaringan utama

Script menggunakan:
1.	File aktif.log untuk menyimpan posisi aktif, 1=utama, 2=cadangan, jadi sebelum script dijalankan perlu dibuat file aktif.log yang berisi nilai 1, echo "1">aktif.log
2.	File jaringan.log untuk menyimpan log setiap adanya perubahan GW

Demikian gambaran pemikiran saya, silakan dikembangkan
Lebih lanjut.
```
#! /bin/bash
targetip="www.detikfdsfasdf.com"

now="$(date)"
aktif="`cat aktif.log`"

ipgwutama="xxx.xxx.xxx.xxx"
nicutama="enp4s0"

ipgwcadangan="xxx.xxx.xxx.xxx"
niccadangan="enp8s0"

#coba ping melalui interface utama
if ping -c 1 $targetip -I $nicutama &> /dev/null
then
        echo "jaringan utama OK"

        #jika gw aktif di jaringan cadangan, maka perlu dikembalikan ke jaringan utama
        if [ "$aktif" == "2" ]
        then
                echo "$now jaringan utama sambung, diputuskan gw dikembalikan ke jaringan utama" >> jaringan.log
                #route del default
                #route add default gw $ipgwutama $nicutama
                echo "1" > aktif.log
        fi
else
        echo "$now jaringan utama FAILED"

        if ping -c 1 $targetip -I $niccadangan &> /dev/null
        then
                echo "$now jaringan cadangan OK"

                #jika gw aktif di jaringan utama, maka perlu diganti ke jaringan cadangan
                if [ "$aktif" == "1" ]
                then
                        echo "$now jaringan utama putus dan jaringan cadangan sambung, diputuskan gw diganti ke jaringan cadangan" >> jaringan.log
                        #route del default
                        #route add default gw $ipgwcadangan $niccadangan
                        echo "2" > aktif.log
                fi
        else
                echo "$now jaringan cadangan juga FAILED"
                #tidak ada hal yang dapat diputuskan, karena jaringan utama maupun cadangan putus
        fi
fi
```

## Menungkinkan ping dari masing-masing interface

ip rule add from $IP1 lookup LINTAS
ip rule add from $IP2 lookup TELKOM
