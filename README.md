Nginx Stream and Elastic Beanstalk

Enabling Nginx Stream for TCP pass through on the instance



Nginx configuration that is bundled with Elastic Beanstalk is setup by default to only handle HTTP traffic. This is evident when looking at the main /etc/nginx/nginx.conf file. The HTTP block encapsulates all other child configuration blocks. The hierarchy looks something like the following:



Nginx.conf =>

http {  includes…

                server { …

                   location { …

                                  } } }



The above hierarchy means that any configuration files including under the /etc/nginx/conf.d/ are encapsulated by the http block, thereby forcing http communication for any listeners configured thereby.


Certain use cases require usage of TCP pass through or usage of Nginx as an instance level load balancer, load balancing between multiple application listeners or containers. The support for this particular Nginx functionality is slightly hidden at the moment. By default Nginx does not support the “Stream” module as this module is configured as “dynamic” with the bundled build of Nginx in Beanstalk. However, thanks to the build flag “--with-stream=dynamic" this module can be installed an enabled using ebextensions.  


Following are the installation steps to enable this module:


     1)    Install the module:

$ sudo yum install nginx-mod-stream.x86_64
2)    Configure Nginx to support the module: Add following line to /etc/nginx/nginx.conf:

load_module  /usr/lib64/nginx/modules/ngx_stream_module.so;

$ sudo service nginx restart

Example: Special Use case:



Customer would like to run a public Docker.io image such as ‘mysql’. Although containerizing MySQL databases is not a good idea, it can be a valid option for temporary read replicas for application testing or Jenkins builds.



For this type of a use case, where a MySQL container is being run on a Single Docker Container Elastic Beanstalk environment, in order to connect to the database remotely, a TCP connection needs to be established between a remote MySQL client such as MySQL Workbench or Linux MySQL agent. With Nginx being configured to only pass HTTP traffic by default, communication between remote host and the container will fail.



In this situation it is desirable to configure the Elastic Beanstalk instance to have a TCP pass through functionality enabled on Nginx. For this example I have used a standard Dockerrun.aws.json which uses the “mysql” image from Docker.io. The ebextensions config and Dockerrun files have been attached to the article



With the above configuration, the user should be able to connect to the container using the CNAME of the Elastic Beanstalk environment:



$ mysql –h <Beanstalk/ELB URL> -P 80 –u root –p 


For this particular use case following two extra steps need to be taken:



1)    If a load balancer is involved, enable TCP pass through



2)    If “…reading initial communication…” error occurs perform the following steps:



Log in to the container:

$ sudo docker exec -i -t <container_id> /bin/bash
Log in to MySQL:

$ mysql –h localhost –u root –p 
Run the query:

FLUSH HOSTS;
