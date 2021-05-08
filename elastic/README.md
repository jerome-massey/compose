## Instructions

1. Copy create-certs.yml, .evn, and instances.yml into your home directory.
2. Run 'docker-compose -f create-certs.yml run --rm create_certs'
3. In Portainer go to 'Stacks' (left side menu) and click 'Add stack', name the stack 'elasticstack' then paste the contents of 'elastic-docker-tls-single.yml' into the 'Web editor'.
4. Add the following environment variables COMPOSE_PROJECT_NAME=elasticstack, CERTS_DIR=/usr/share/elasticsearch/config/certificates, VERSION=7.12.1
5. Deploy the stack.
6. Console into the elastic search container (es01) and run the following command 'bin/elasticsearch-setup-passwords auto --batch --url https://es01:9200'(!!IMPORTANT!!) you must grab the passwords that are generated.
7.  Under 'Stacks' edit the config in the editor and change the ELASTICSEARCH_PASSWORD: in the stack to the password you saved then 'Update the stack'.




## Filebeat



https://www.elastic.co/guide/en/beats/filebeat/current/privileges-to-publish-events.html
https://www.elastic.co/guide/en/beats/filebeat/current/privileges-to-setup-beats.html

1. Create users and roles from the links listed "filebeat_writer" and "filebeat_kib_setup"
2. Create /beats/filebeat.yml file and paste the below in the file. Also create /root/beats/filebeat/prospectors.d/ directory.
2. Create another stack and paste elastic-docker-fb.yml add the following environement variables (ELASTICSEARCH_HOST=https://es01:9200 , ELASTICSEARCH_USERNAME=filebeat_writer , ELASTICSEARCH_PASSWORD=CHANGEME , KIBANA_HOST_USERNAME=filebeat_kib_setup , KIBANA_HOST=kib01:5601 , KIBANA_PASSWORD=CHAMGEME , VERSION=7.12.1)
3. Deploy the stack

- After the container comes up edit the config file adding and place in in /root/beats/ on the host and create a directory in /root/beats/ on the host named prospectors.d

filebeat.config:
  modules:
    path: ${path.config}/modules.d/*.yml
    reload.enabled: false

processors:
  - add_cloud_metadata: ~
  - add_docker_metadata: ~

output.elasticsearch:
  hosts: '${ELASTICSEARCH_HOST}'
  username: '${ELASTICSEARCH_USERNAME}'
  password: '${ELASTICSEARCH_PASSWORD}'
  ssl.enabled: true
  ssl.verification_mode: none

setup.kibana:
  host: '${KIBANA_HOST}' 
  username: '${KIBANA_HOST_USERNAME}'
  password: '${KIBANA_PASSWORD}'
  ssl.enabled: true
  ssl.verification_mode: none