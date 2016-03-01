# bitesize-training notes

All credit due goes to mward29 and the kubernetes source team for providing this mysql-wordpress example.

### Prequisities 
  A bitesize cluster running k8s. Preferably your own namespace.

### Files
  Files are mentioned in sequence they should applied
  
    * wp-namespace.yaml
      * creates new k8s namespace
    * wordpress-service.yaml
      * k8s svc file for 80 - wordpress/php/apache foreground mode
    * mysql-service.yaml
      * k8s svc file for 3306 - mysqld
    * mysql.yaml
      * k8s pod file for mysqld
    * wordpress.yaml
      * k8s pod file for wordpress/php/apacheF
    * nginx-svc.yaml
      * k8s svc for nginx 
    * nginx-ingress.yaml
      * nginx replication controller - reverse proxy to internal services.
    * ingress.yaml
      * k8s public ingress def
    
    
### Deploy

All kubernetes definition files can be applied with `kubectl create -f <file.yaml> <--namespace=<ns_name>>`

1. Create namespace
  * `kubectl create -f wp-namespace.yaml`
1. Verify
  * `kubectl get ns`
1. Create service expositions
  * `kubectl --namespace=wordpressns create -f wordpress-svc.yaml`
  * `kubectl --namespace=wordpressns create -f mysql-svc.yaml`
1. Create pods 
  * `kubectl --namespace=wordpressns create -f mysql.yaml`
  * `kubectl --namespace=wordpressns create -f wordpress.yaml`
1. If desire one can verify wordpress established proper connection to mysql
  * `kubectl --namespace=wordpressns get po`
  * `kubectl --namespace=wordpressns logs <pod name>`
  * `kubectl --namespace=wordpressns describe po`
1. Create nginx-svc (this can actually be done when you create the other services\
  be sure the selectors match up to the service you are mapping to)
  * `kubectl --namespace=wordpressns create -f nginx-svc.yaml`
1. Create nginx-ingress
  * `kubectl --namespace=wordpressns create -f nginx-ingress.yaml`
  * notice nodeSelector: role: loadbalancer. run `kubectl get no` to inspect.
  * the nginx-ingress will reside on nodes labeled loadbalancer. Note in ec2console they have public EIPs.
  * you will obtain and use these EIPs when creating your public route53 name
1. Create k8s ingress
  * this generates the nginx server config that maps <hostname> -> <k8s svc>
  * modify `spec/rules/host` to eq. your public hostname you will create in route53 prsn.io hosted zone
  * `kubectl --namespace=wordpressns create -f ingress.yaml`
1. Create <hostname>.prsn.io in route53 console with A record to point at the 3 EIPs
1. PROFIT - visit <hostname>.prsn.io

