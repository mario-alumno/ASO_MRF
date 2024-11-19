```bash
#!/bin/bash
read -p 'Introduce una direccion IP en CIDR: ' CIDR

red=$(echo $CIDR | cut -d '/' -f 1)
mascara=$(echo $CIDR | cut -d '/' -f 2)

IFS='.' read -r -a parte_ip <<< '$red'
case $mascara in
    24) RANGE=254;;
    16) RANGE=65535;;
     8) RANGE=16777215;;
     *) echo 'Esta mascara no existe'; exit 1;;
esac


read -p 'Nombre del archivo donde quieres guardar la informacion: ' archivo
for ip in $(seq 1 $RANGE);
do
   ip_escaneada=$CIDR
   if ping -c 1 -W 1 $ip_escaneada &>/dev/null
   then
      echo "$ip_escaneada"
      ttl=$(ping -c 1 -W 1 $ip_escaneada | grep ttl | awk -F 'ttl=' '{print $2}' | awk '{print $1}')        
      if [ '$ttl'-eq 64 ]
      then
        echo OS='Linux'
      elif [ '$ttl' -eq 128 ]
      then
         echo OS='Windows'
      else
         echo OS='No conozco el sistema operativo'
      fi
     puertos=' '
     for puerto in 22 80 443
     do
        nc -zv -w 1 $ip_escaneada $puerto &>/dev/null && puertos+='$puerto'
     done
     echo 'Escaneo de la red' >  $archivo
     informacion=$('IP: $ip_escaneada | OS: $OS | Puertos: $puertos')
     echo=$informacion

  fi
done
echo Todos los resultados han sido almacenados en $archivo
echo $informacion
```