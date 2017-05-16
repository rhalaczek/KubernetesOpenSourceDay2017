## **Warsztaty** Orkiestracja kontenerów z Kubernetes Open Source Day 2017

Repozytorium przchowujące konfigurację środowiska warsztatowego
na potrzeby warsztatów Orkiestracja kontenerów z Kubernetes prowadzonych
w trakcie Open Source Day 2017

### Przygotowanie

Zaloguj się na host master swojego klastra Kubernetes:
	
	- FQDN:
		Każdemu użytkownikowi zostal przydzielony osobny klaster.
		- 01-25  lpkubelaba<NUMER>mgmt.northeurope.cloudapp.azure.com
			Login: userkubelaba<NUMER>
		- 26-50	lpkubelabb<NUMER>mgmt.northeurope.cloudapp.azure.com
			-Login: userkubelabb<NUMER> 
	- [Klucz prywatny] (https://www.dropbox.com/s/g2of30pq01v5b1r/kubernetes_rsa_id?dl=0) - hasło zostanie przekazane przez instruktora
		- zmień uprawnienia do klucza
```
chmod 600 kubernetes_rsa_id
```

### Ćwiczenie

Zaloguj się do serwera pełniącego role mastera w klastrze Kubernetes:

```
ssh userkubelaba01@lpkubelaba01mgmt.northeurope.cloudapp.azure.com -i ./Klucze/kubernetes_rsa_id
```

#### Przygotuj środowisko do pracy:

```
echo "source <(kubectl completion bash)" >> ~/.bashrc
source <(kubectl completion bash)
mkdir ~/warsztaty
cd ~/warsztaty
git clone https://github.com/rhalaczek/KubernetesOpenSourceDay2017
cd ./KubernetesOpenSourceDay2017 
```

Przygotuj interfejs webowy 
	- na serwerze master klastra:

```
kubectl proxy --port=8443 &
exit
```

	- na lokalnej maszynie

```
ssh -L 2222:localhost:8443 -N userkubelaba01@lpkubelaba01mgmt.northeurope.cloudapp.azure.com -i ./Klucze/kubernetes_rsa_id &
```

Uruchom przeglądarkę :
http://localhost:2222/ui

####Rozpocznij ćwiczenie

#####Instalacja instancji bazy danych MySQL:

Ponownie zaloguj do serwera pełniącego role mastera w klastrze Kubernetes:

```
ssh userkubelaba01@lpkubelaba01mgmt.northeurope.cloudapp.azure.com -i ./Klucze/kubernetes_rsa_id
```

Dodaj persistent volumes do klastra:

```
kubectl  create -f mysql_persistent.yaml
```

Sprawdź poprawność wykonania

```
kubectl get pvc
kubectl get pv
```

Dodaj instancję serwera MySQL:

```
kubectl create mysql.yaml
```

Sprawdź poprawność instalacji :

```
kubectl get pod
kubectl get deployment
```

Sprawdź adres ip bazy MySQL i zaloguj się do bazy

```
kubectl get pod -o wide
kubectl run mysql-client --image=mysql:5.6 -i -t --rm --restart=Never --  mysql -h [ip poda] -u root -ppassword
```

Sprawdź poprawność działania bazy danych:

```
mysql> SHOW DATABASES;
mysql> CREATE DATABASE testABC;
mysql> USE testABC;
mysql> CREATE TABLE nazwa(nazwa varchar(50));
mysql> INSERT INTO nazwa(nazwa) VALUES ('test A'),('test B'),('TEST c');
mysql> SELECT nazwa FROM nazwa;
mysql> exit;
```

Usuń pod mysql i sprawdź poprawnośc działania bazy danych:

```
kubectl get pods
kubectl delete pod [nazwa poda]
kubectl get pod -o wide
kubectl run mysql-client --image=mysql:5.6 -i -t --rm --restart=Never --  mysql -h [ip poda] -u root -ppassword
mysql> SHOW DATABASES;
mysql> USE testABC;
mysql> SELECT nazwa FROM nazwa;
mysql> exit;
```

Uruchom serwis udostępniający usługę MySQL:

```
kubectl create -f mysql_service.yaml
```

Sprawdź konfigurację usługi:

```
kubectl get svc
kubectl run mysql-client --image=mysql:5.6 -i -t --rm --restart=Never --  mysql -h [ip service] -u root -ppassword
mysql> SHOW DATABASES;
```

Korzystając z drugiej konsoli (na pierwszej pozostaw aktywnego klienta MySQL) zaloguj się ponownie do klastra:

```
ssh userkubelaba01@lpkubelaba01mgmt.northeurope.cloudapp.azure.com -i ./Klucze/kubernetes_rsa_id
```

Usuń pod MySQL:

```
kubectl get pod
kubectl delete pod [nazwa poda]
```

Powróć do konsoli z klientem MySQL:

```
mysql> SHOW DATABASES;
mysql> exit;
```

Poprawność i zasadność używania obiektu typu 'service' zostala potwierdzona.


#####Instalacja instancji wordpressa:

```
kubectl create -f wordpress.yaml
```

Zainstaluj usługę serwującą aplikację worpress na świat:

```
kubectl create -f wordpress_service.yaml
```

Sprawdź publiczny adres IP wystawionej usługi:

```
kubectl get svc
```

Sprawdź na lokalnym komputerze, w trybie graficznym połączenie z przeglądarką z 
[EXTERNAL-IP] usługi wordpress

Przeprowadź podstawową konfigurację usługi worpdress
**WAŻNE!** skopiuj uzytkownika i hasło - będą jeszcze potrzebne

Usuń pod wordpress: 

```
kubectl get pod
kubectl delete pod [nazwa poda]
```

Przeprowadź update aplikacji wordpress do wersji 4.7.4 za pomocą panelu administracyjnego wordpress.

Zrestartuj pod wordpress:

```
kubectl get pod
kubectl delete pod [nazwa]
```

Zaloguj się ponownie do panelu sterowania aplikacji wordpress i sprawdź wersję aplikacji.


Dodaj persistent volune do kontenera, w którym uruchomiona jest aplikacja wordpress:

```
kubectl create -f wordpress_persistent.yaml
```

Podłącz persistent volume do aplikacji wordpress:

```
kubectl edit deployment wordpress

        volumeMounts:
        - name: wordpress-persistent-storage
          mountPath: /var/www/html

      volumes:
      - name: wordpress-persistent-storage
        persistentVolumeClaim:
          claimName: wp-pv-claim
```

Zaloguj się ponownie do panelu sterowania aplikacji wordpress i przeprowadź
update aplikacji do wersji 4.7.4

Zrestartuj pod z aplikacją wordpress:

```
kubectl get pod
kubectl delete pod [nazwa poda]
```

Ponownie przejdź do panelu sterowania aplikacji wordpress, jakie zauważyłeś/aś
różnice w stusunku do poprzedniej operacji restartu poda z aplikacją wordpress?


#####Dodatkowe komendy dla wytrwałych:

```
kubectl describe svc wordpress
kubctl logs [nazwa poda]
kubectl exec -it [nazwa poda] -- /bin/bash
printenv
```

