######
Meetup
######

Create swarm
------------

  ..  code::

      docker-machine create -d virtualbox ops1
      docker-machine create -d virtualbox ops2
      docker-machine create -d virtualbox ops3

  ..  code::

      eval $(docker-machine env ops1)
      docker swarm init --advertise-addr 192.168.99.100
      Swarm initialized: current node (d07w3nifvzwb5dmfykw8d0p88) is now a manager.
      To add a worker to this swarm, run the following command:

          docker swarm join \
          --token SWMTKN-1-5637fsoo3iavhazcwywhldxi5jeuw61i7nvfrybgxherazphjn-3h6tyc12kzy960p66qz8hds9v \
          192.168.99.100:2377

      To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

  ..  code::

      eval $(docker-machine env ops2)
      docker swarm join \
      --token SWMTKN-1-5637fsoo3iavhazcwywhldxi5jeuw61i7nvfrybgxherazphjn-3h6tyc12kzy960p66qz8hds9v \
      192.168.99.100:2377

  ..  code::

      eval $(docker-machine env ops3)
      docker swarm join \
      --token SWMTKN-1-5637fsoo3iavhazcwywhldxi5jeuw61i7nvfrybgxherazphjn-3h6tyc12kzy960p66qz8hds9v \
      192.168.99.100:2377

Create network
--------------

  ..  code::

      eval $(docker-machine env ops1)
      docker network create -d overlay dockercoins

  ..  code::

      docker service create --name rng --network dockercoins --mode global jpetazzo/dockercoins_rng:1465439244


Debug overlay network
---------------------


  ..  code::

      docker service create --network dockercoins \
      --name debug --mode global \
      alpine sleep 1000000000

  ..  code::

      / # drill rng
      ;; ->>HEADER<<- opcode: QUERY, rcode: NOERROR, id: 868
      ;; flags: qr rd ra ; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0
      ;; QUESTION SECTION:
      ;; rng.	IN	A

      ;; ANSWER SECTION:
      rng.	600	IN	A	10.0.0.2

      ;; AUTHORITY SECTION:

      ;; ADDITIONAL SECTION:

      ;; Query time: 0 msec
      ;; SERVER: 127.0.0.11
      ;; WHEN: Mon Nov 14 20:36:45 2016
      ;; MSG SIZE  rcvd: 40


Rolling update
--------------

  .. code::

     eval $(docker-machine env ops1)

     docker tag jpetazzo/dockercoins_rng:1465439244 ballot/dockercoins_rng:latest

     watch -n1 "docker service ps rng | grep -v Shutdown.*Shutdown"

     docker service update rng --image ballot/dockercoins_rng:latest


ELK
---

  .. code::

     docker network create --driver overlay logging

     docker service create --network logging --name elasticsearch elasticsearch:2.4

     docker service create --network logging --name kibana \
     --publish 5601:5601 \
     -e ELASTICSEARCH_URL=http://elasticsearch:9200 kibana:4.6

     docker service create --network logging \
     --name logstash -p 12201:12201/udp \
     logstash:2.4 -e "$(cat /home/ballot/git/training/docker_ops_intermediate/logstash.conf)"
