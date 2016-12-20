Prereqs:
VM Box (I am using Oracle)
bash shell (Git by me)

Vagrant:
swarm client (66.55.44.13), swarm manager(66.55.44.10), swarm node1(66.55.44.11), swarm node2(66.55.44.12)
port 2375 for swarm manager, swarm node1 (with swarm manager fallback) and swarm node2 has to be open for docker listener


docker:
consul (66.55.44.10):
docker run --restart=always -d -p 8500:8500 -p 172.17.0.1:53:8600/udp -p 8400:8400 gliderlabs/consul-server -node myconsul -bootstrap

 - Within Consul, port 8400 is used for RPC, 8500 is used for HTTP, 8600 is used for DNS. By using “-p” option, we are exposing these ports to the host machine.
 - "172.17.0.1” is the Docker bridge IP address. We are remapping Consul Container’s port 8600 to host machine’s Docker bridge port 53 so that Containers on that host can use Consul for DNS.
 - “bootstrap” option is used to operate Consul in standalone mode.


swarm manager:
(66.55.44.10)
docker run --restart=always -d -p 3375:2375 swarm manage --replication --advertise 66.55.44.10:8500 consul://66.55.44.10:8500
do not forget: 2375 docker container context, 3375 swarm manager context
(66.55.44.11)
docker -H=tcp://66.55.44.11:2375 run --restart=always -d -p 3375:2375 swarm manage --replication --advertise 66.55.44.11:8500 consul://66.55.44.10:8500


swarm node1 (66.55.44.11):
docker -H=tcp://66.55.44.11:2375 run -d --restart=always swarm join --advertise 66.55.44.11:2375 consul://66.55.44.10:8500

swarm node2 (66.55.44.12):
docker -H=tcp://66.55.44.12:2375 run -d --restart=always swarm join --advertise 66.55.44.12:2375 consul://66.55.44.10:8500


nginx1, nginx2 (deploy via swarm manager and register to consul - http1.json, http2.json)
docker -H=tcp://66.55.44.11:2375 run --restart=always -d -p 80:80 --name nginx1 nginx
docker -H=tcp://66.55.44.12:2375 run --restart=always -d -p 80:80 --name nginx2 nginx

curl -X PUT --data-binary @/vagrant/http1.json http://66.55.44.10:8500/v1/agent/service/register
curl -X PUT --data-binary @/vagrant/http2.json http://66.55.44.10:8500/v1/agent/service/register


ubuntu docker container as a client (with apache as a proxy )(deploy as a single-node docker container on 66.55.44.10)
docker -H=tcp://66.55.44.10:2375 run --restart=always -d -ti -p 82:80 smakam/myubuntu:v3 bash
--and try curl http://http.service.consul:80
install apache2
modify /etc/apache2/apache2.conf
ProxyPass "/nginx" "http://http.service.consul"
ProxyPassReverse "/nginx" "http://http.service.consul"
ProxyPass "/myapp" "http://myapp.service.consul:38090"
ProxyPassReverse "/myapp" "http://myapp.service.consul:38090"
a2enmod proxy_http
service apache2 stop
service apache2 start



ENJOY ;-)


docker -H=tcp://66.55.44.11:2375 run --restart=always -d  -p 38090:38090 -v /vagrant:/vagrant --name myapp1 cogniteev/oracle-java java -jar /vagrant/static-inventory-loader-app-0.0.1-SNAPSHOT.jar
docker -H=tcp://66.55.44.12:2375 run --restart=always -d  -p 38090:38090 -v /vagrant:/vagrant --name myapp2 cogniteev/oracle-java java -jar /vagrant/static-inventory-loader-app-0.0.1-SNAPSHOT.jar
curl -X PUT --data-binary @/vagrant/myapp1.json http://66.55.44.10:8500/v1/agent/service/register
curl -X PUT --data-binary @/vagrant/myapp2.json http://66.55.44.10:8500/v1/agent/service/register