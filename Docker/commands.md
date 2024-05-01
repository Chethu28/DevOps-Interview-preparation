1. **yum install docker -y** -------> to install docker
2. **docker version** ------->  to check the docker version
3. **systemctl status docker** ------->  to check the docker status
4. **systemctl start docker** ------->  to start the docker
5. **docker images** ------->  used to get list of images in our server
6. **docker run image_name** ------->  (ubuntu) to get the image
7. **docker ps -a** ------->  used to see all the containers
8. **docker run -it --name cont_name image_name** ------->  used to create a container
9. **docker attach cont_name/id** ------->   to enter into the container
10. **docker stop cont_name/id** ------->   to stop the container
11. **docker start cont_name/id** ------->   to start the docker
12. **docker stop $(docker ps -a -q)** ------->   to stop all containers
13. **docker start $(docker ps -a -q)** ------->   to start all containers
14. **docker rm $(docker ps -a -q)** ------->   to delete all containers
15. **docker rm cont_name/id** ------->   to delete single container
16. **docker rmi image_name/id** ------->   used to delete image
17. **ctlr+p+q** ------->   used to exit from the container
18. **docker run -it -d --name cont_name image_name** ------->   used to create a container (runs on background also)
19. **docker run -it -d --name cont_name -p 8081:80 image_name** ------->   used to create a container with port number (8081 --> host port, 80 ---> container port)
20. **docker exec -it cont_name "command"** ------->   used to perform the commands inside the container
21. **docker contianer prune** ------->   used to remove unused containers
22. **docker ps** ------->   used to see running containers only
23. **docker ps -a -f "status=exited"** ------->   used to see only stopped containers
24. **docker container ls** ------->   used to see running containers only
25. **docker container ls -a** ------->   used to see all the containers
26. **docker container ls -a -n 2** ------->   used to see latest 2 containers
27. **docker container ls --latest** ------->   used to see the latest container
28. **docker container ls -a -s** ------->   used to see all the containers along with the sizes
29. **docker commit cont_name image_name** ------->   create image from the container
30. **docker rename old_cont_name new_cont_name** ------->   used to rename the container
31. **docker build -t image_name .** ------->   to build the docker file (. represents the path of the docker file)
32. **docker build -t image_name:tag_name .** ------->   to build the docker file with tag
33. **docker volume create volume_name** ------->   used to create a volume
34. **docker volume rm volume_name** ------->   used to delete the volume
35. **docker volume ls** ------->   used to see the list of volumes
36. **docker volume inspect volume_name/id** ------->   used to inspect volume
37. **docker volume prune** ------->   used to delete unused volumes
38. **docker run -it --name cont_name -v /volume_name image_name** ------->   used to create a volume inside a container
39. **docker run -it --name cont_name --privileged=true --volumes-from cont_name image** -------> used to share the volumes to container 
40. **docker run -it --name cont_name --mount source=volume_name,destination=volume** -------> used to mount the local volumes to container volumes
41. **docker run -it --name cont_name -v local_volume:/cont_volume image_name** ------->   base volume shared to container
42. **docker run -it --name cont_name -v $("pwd"):/volume_name image_name** ------->    used directory as a volume inside the container
43. **docker inspect cont_name/id** ------->   used to get the details of the container
44. **docker image inspect image_name/id** ------->   used to get the detail of the image
45. **docker network ls** ------->   used to get the list of networks
46. **docker network create network_name** ------->   used to create a network
47. **docker network connect network_name cont_name** ------->   used to attach a network to container
48. **docker network disconnect network_name cont_name** ------->   used to disconnect the network from the container
49. **docker network rm network_name** ------->   used to remove network
50. **docker network prune** ------->   used to remove unused networks
51. **docker run -it --name cont_name --network network_name image_name** ------->   used to create a container along with the network
52. **docker login** ------->   used to login into dockerhub
53. **docker tag image_name docker_hub_id/repo_name** ------->  used to tag the image to docker hub  
54. **docker push docker_hub_id/repo_name** ------->   used to push the image from local to docker hub
55. **docker pull image_name** ------->   used to get the image from docker hub to local
56. **docker swarm init --advertise-addr public_ip_of_manager** ------->   used to generate a token for adding worker nodes to swarm
57. **docker swarm join-token manager** ------->   generate the manager token
58. **docker swarm join-token worker** ------->   generates the worker token
59. **docker node ls** ------->   used to see the list of nodes
60. **docker service create --name service_name --replicas 2 --publish 8081:80 image_name** ------->   used to create a service
61. **docker service ls** ------->   used to see list of services
62. **docker service rm service_name** ------->   used to delete service
63. **docker service update --image service_name** ------->   used to update the image of the service
64. **docker service rollback service_name** ------->   used to rollback to prev service
65. **docker service scale service_name=4** ------->   used to increase the replicas
66. **docker service inpsect service_name** ------->   to inspect a service
67. **docker service logs service_name** ------->   used to get logs of a service 
68. **docker-compose up -d** ------->   used to execute the compose file
69. **docker-compose down** ------->   remove the containers
70. **docker-compose stop** ------->   used to stop the compose containers
71. **docker-compose start** ------->   used to start the containers
72. **docker-compose pause** ------->   used to pause the containers
73. **docker-compose unpause** ------->   used to unpause the containers
74. **docker-compose ps** ------->   used to get the compose containers
75. **docker-compose build** ------->   used to build the images which are present in compose file
76. **docker-compose images** ------->   used to get the images of the compose files
77. **docker-compose logs** ------->   used to get the logs of compose file
78. **docker-compose config** ------->   used to get the details of the compose file
79. **docker-compose -f file_name.yml up -d** ------->   execute any compose file
80. **docker stack deploy --config-name docker-compose.yml stack_name** ------->   used to deploy a stack file
81. **docker stack ls** ------->   used to get list of stacks
82. **docker stack ps stack_name** ------->   used to get the containers associated to our stack
83. **docker stack services stack_namme** ------->   used to get the services for our stack
84. **docker stack rm stack_name** ------->   used to remove the stack
85. **docker system df -v** ------->   used to see all the docker components information