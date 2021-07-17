# Acesse seu roteador Claro pelo Wi-Fi

Entre em http://192.168.0.1/

Descubra a senha na etiqueta do modem.

https://blog.clarocombomais.com.br/como-mudar-senha-do-roteador/

# Redirecionando o Tráfego NAT

https://www.iperiusbackup.net/pt-br/aprenda-a-redirecionar-portas-para-a-internet/

Normalmente as portas de entrada conhecidas como comerciais a exemplo da 21, 23, 25, 80 e 110 costumam ser bloqueadas pelo provedor em planos de internet doméstica, por este motivo é aconselhado utilizar outras combinações que fazem alusão à elas por exemplo a porta 808 como referência para a porta 80.

Neste exemplo estando ativo um servidor HTTP (ou Web) no seu computador doméstico com o IP 192.16810.10 mas em razão da limitação comentada, no navegador será digitado 221.34.50.214:808 onde o roteador fará o redirecionamento para o endereço interno do computador configurado como servidor web. Caso tenha um servidor FTP em outro computador com o IP 192.168.10.25 a porta de endereço digitado na aplicação mudará para 221.34.50.214:2121 (em alusão à porta 21), caso mais serviços estejam ativos num mesmo computador será suficiente configurar no roteador a respectiva porta do serviço a ser disponibilizado. Reforçando o exemplo acima do computador doméstico com IP interno 192.168.10.25, executando um servidor WEB, FTP e MINECRAFT onde após configurando o redirecionamento de portas no roteador teremos o seguinte cenário:

WEB – (internet)221.34.50.214:8080 – (rede local)192.168.10.25

FTP – (internet)221.34.50.214:2121 – (rede local)192.168.10.25

MINECRAFT – (internet)221.34.50.214:25565 – (rede local)192.168.10.25

