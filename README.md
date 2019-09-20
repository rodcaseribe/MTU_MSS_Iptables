# MTU_MSS_Iptables - PPPOE Linux
MTU_MSS_Iptables

# Comandos ativar instalar e ativar pppoe

apt-get install -y ppp pppoeconf

pppoeconf eth6

# Verificar rota se está na MAIN

ip route show table main

# Ativar Link

pon dsl-provider

# Desativar Link 

poff -a

# Configuracao de /etc/ppp/peers/dsl-provider

noipdefault

defaultroute

replacedefaultroute

hide-password

lcp-echo-interval 10

lcp-echo-failure 3

noauth

persist

noaccomp

default-asyncmap

mtu 1500

maxfail 0

holdoff 5

plugin rp-pppoe.so eth6

user "USUARIO"

usepeerdns

# Comandos Iptables

iptables -t nat -A POSTROUTING -o $WAN -j MASQUERADE

iptables -I FORWARD -p tcp --tcp-flags SYN,RST SYN -j TCPMSS --clamp-mss-to-pmtu

iptables -t mangle -I FORWARD -o ppp+ -p tcp -m tcp --tcp-flags SYN,RST SYN -m tcpmss --mss 1453:65535 -j TCPMSS --set-mss 1452

iptables -t mangle -I FORWARD -i ppp+ -p tcp -m tcp --tcp-flags SYN,RST SYN -m tcpmss --mss 1373:65535 -j TCPMSS --set-mss 1372

# Analizar com Wireshark a fragmentação de Pacotes, fazendo filtragem por MSS

iptables -I FORWARD -p tcp --tcp-flags SYN,RST SYN -m tcpmss --mss 1400:1600 -j TCPMSS --set-mss 1452 

# Liberar http e https

iptables -I FORWARD -p tcp --sport 443 -j ACCEPT

iptables -I FORWARD -p tcp --dport 443 -j ACCEPT

iptables -I FORWARD -p tcp --sport 80 -j ACCEPT

iptables -I FORWARD -p tcp --dport 80 -j ACCEPT

# Desativar fragmentacao da placa LAN tso e gso p/ kernel, verificar WAN

ethtool --show-offload  eth5

ethtool -K eth5 tso off

ethtool -K eth5 gso off

# Verificacoes de firewall MSS

iptables -L -n -v -t mangle | grep mss
