##WARSZTATY Orkiestracja kontenerów z Kubernetes Open Source Day 2017

Repozytorium przchowujące konfigurację środowiska warsztatowego
na potrzeby warsztatów Orkiestracja kontenerów z Kubernetes prowadzonych
w trakcie Open Source Day 2017

####Przygotowanie
Zaloguj się na host master swojego klastra Kubernetes:
	-FQDN:
		Każdemu użytkownikowi zostal przydzielony osobny klaster.
		- 01-25  lpkubelaba<NUMER>mgmt.northeurope.cloudapp.azure.com
			Login: userkubelaba<NUMER>
		- 26-50	lpkubelabb<NUMER>mgmt.northeurope.cloudapp.azure.com
			-Login: userkubelabb<NUMER> 
	-[Klucz prywatny](https://github.com) - hasło zostanie przekazane przez instruktora

##Ćwiczenie
Zaloguj się do serwera pełniącego role mastera w klastrze Kubernetes:

```
ssh userkubelaba01@lpkubelaba01mgmt.northeurope.cloudapp.azure.com -i ./Klucze/id_rsa
```

###Przygotuj środowisko do pracy:

```
echo "source <(kubectl completion bash)" >> ~/.bashrc
source <(kubectl completion bash)"
mkdir ~/warsztaty
cd ~/warsztaty
git clone https://github.com/rhalaczek/KubernetesOpenSourceDay2017 
```

przygotowanie UI do wystawienia na świat
na serwerze master klastra:

```
kubectl proxy --port=8443 &
exit
```

na lokalnej maszynie

```
ssh -L 2222:localhost:8443 -N userkubelaba01@lpkubelaba01mgmt.northeurope.cloudapp.azure.com -i ./Klucze/id_rsa &
```

uruchomienie przeglądarki na localhost :
http://localhost:2222/ui

###Rozpocznij ćwiczenie

ponowne logowanie do serwera pełniącego role mastera w klastrze Kubernetes:

```
ssh userkubelaba01@lpkubelaba01mgmt.northeurope.cloudapp.azure.com -i ./Klucze/id_rsa
```

dodanie persistent volumes do klastra:

```
kubectl  create -f ./v1_mysql_persistent.yaml
```

sprawdzenie poprawności wykonania

```
kubectl get pvc
kubectl get pv
```

dodanie instancji serwera MySQL:

```
kubectl create -f ./v1_mysql.yaml
```

sprawdzenie poprawności instalacji :

```
kubectl get pod
kubectl get deployment
```

sprawdzenie adresu ip bazy MySQL i zalogowanie do bazy

```
kubectl get pod -o wide
kubectl run mysql-client --image=mysql:5.6 -i -t --rm --restart=Never --  mysql -h [ip poda] -u root -ppassword
```

sprawdzenie poprawności działania bazy danych:

```
mysql> SHOW DATABASES;
mysql> CREATE DATABASE testABC;
mysql> USE testABC;
mysql> CREATE TABLE nazwa(nazwa varchar(50));
mysql> INSERT INTO nazwa(nazwa) VALUES ('test A'),('test B'),('TEST c');
mysql> SELECT nazwa FROM nazwa;
mysql> exit;
```

restart poda i sprawdzenie poprawności działania bazy danych

```
kubectl get pods
kubectl delete pod [nazwa poda]
watch 'kubectl get pod -o wide'
kubectl run mysql-client --image=mysql:5.6 -i -t --rm --restart=Never --  mysql -h [ip poda] -u root -ppassword
mysql> SHOW DATABASES;
mysql> USE testABC;
mysql> SELECT nazwa FROM nazwa;
mysql> exit;
```

uruchomienie usługi serwujacej usługe MySQL

```
kubectl create -f ./v1_mysql_service.yaml
```

sprawdzenie konfiguracji usługi:

```
kubectl get svc
kubectl run mysql-client --image=mysql:5.6 -i -t --rm --restart=Never --  mysql -h [ip service] -u root -ppassword
mysql> SHOW DATABASES;
```

na drugiej konsoli (na pierwszej zostaje aktywny klient MySQL) logowanie do klastra:

```
ssh userkubelaba01@lpkubelaba01mgmt.northeurope.cloudapp.azure.com -i ./Klucze/id_rsa
```

restart poda:

```
kubectl get pod
kubectl delete pod [nazwa poda]
```

powrót na konsolę z klientem MySQL:

```
mysql> SHOW DATABASES;
mysql> exit;
```

Poprawność i zasadność używania obiektu typu 'service' zostala potwierdzona.


instalacja instancji wordpressa:

```
kubectl create -f ./v1_wordpress.yaml
```

instalacja usługi serwujacej aplikację wordpress na świat:

```
kubectl create -f ./v1_wordpress_service.yaml
```

sprawdzenie zewnętrznego adresu ip usługi:

```
kubectl get svc
```

na lokalnym komputerze, w trybie graficznym połączenie z przeglądarką z 
[EXTERNAL-IP] usługi wordpress

przeprowadzić podstawową konfigurację usługi worpdress
restart poda:

```
kubectl delete pod [nazwa poda]
```

przeprowadzić update wordpressa do wersji 4.7.4 z panelu sterowania
restart poda
zalogować się ponownie do panelu sterowania wordpressa


dodanie persistent volume do wordpresa

```
kubectl create -f ./v1_wordpress_persistent.yaml
```

podłączenie persistent volume do instancji wordpressa:

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

zalogowac się do panelu sterowania wordpress
przeprowadzić update do wersji 4.7.4

restart poda:

```
kubectl get pod
kubectl delete pod [nazwa poda]
```

Sprawdź panel sterowania swojej instancji wordpressa



dodatkowe komendy

```
kubectl describe svc wordpress
kubctl logs [nazwa poda]
kubectl exec -it [nazwa poda] -- /bin/bash
printenv
```

