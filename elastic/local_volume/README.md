## Instructions

1. Copy create-certs.yml, .evn, and instances.yml into your home directory.
2. Run 'docker-compose -p elasticstack -f create-certs.yml run --rm create_certs'
3. In Portainer go to 'Stacks' (left side menu) and click 'Add stack', name the stack 'elasticstack' then paste the contents of 'elastic-docker-tls-single.yml' into the 'Web editor'.
4. Add the following environment variables COMPOSE_PROJECT_NAME=elasticstack, CERTS_DIR=/usr/share/elasticsearch/config/certificates, VERSION=7.12.1
5. Deploy the stack.
6. Console into the elastic search container (es01) and run the following command 'bin/elasticsearch-setup-passwords auto --batch --url https://es01:9200'(!!IMPORTANT!!) you must grab the passwords that are generated.
7.  Under 'Stacks' edit the config in the editor and change the ELASTICSEARCH_PASSWORD: in the stack to the password you saved then 'Update the stack'.




## Filebeat

Install with package manager.
https://www.elastic.co/guide/en/beats/filebeat/7.12/setup-repositories.html


https://www.elastic.co/guide/en/beats/filebeat/current/privileges-to-publish-events.html

https://www.elastic.co/guide/en/beats/filebeat/current/privileges-to-setup-beats.html


1. Create users and roles from the links listed "filebeat_writer" and "filebeat_setup"
  - Also add 'cluster:admin/ilm/put' to Cluster privileges - Needed to run 'filebeat setup -e'
2. Edit filebeat.yml

filebeat.config:
    reload.enabled: false

output.elasticsearch:
    hosts: https://<host>:9200
      username: filebeat_writer
      password: '${FBW_PWD}'
      ssl.enabled: true
      ssl.verification_mode: none

setup.kibana:
  host: https://<host>:5601 
  username: filebeat_setup
  password: '${FBS_PWD}'
  ssl.enabled: true
  ssl.verification_mode: none


##  Grok Patterns to apply to filebeat-7.12.1-cisco-ios-pipeline

%{SYSLOG5424PRI}(%{NUMBER:log_sequence})?:( %{HOSTNAME}:)? %{CISCOTIMESTAMPTZ:log_date}: %%{CISCO_REASON:facility}-%{INT:severity_level}-%{CISCO_REASON:facility_mnemonic}: %{GREEDYDATA:message}

%{SYSLOG5424PRI}(%{NUMBER:log_sequence})?:( %{HOSTNAME}:)? %{CISCOTIMESTAMPTZ:log_date}: %%{CISCO_REASON:facility}-%{CISCO_REASON:facility_sub}-%{INT:severity_level}-%{CISCO_REASON:facility_mnemonic}: %{GREEDYDATA:message}

# Pattern definitions

{
  "CISCOTIMESTAMPTZ": "%{CISCOTIMESTAMP}( %{TZ})?"
}



# Cisco Configs for Logging.

ip name-server {{ PRI_DNS }} {{ SEC_DNS }}
!
ntp server {{ NTP_SERVER }}
!
clock timezone EDT -4 0
!
service timestamps log datetime localtime
service timestamps debug datetime localtime
!
archive
log config
logging enable
logging size {{ NUMBER }}
hidekeys
notify syslog
!
logging trap notifications
logging origin-id hostname
logging host {{ IP }} transport udp port {{ PORT }}
!
archive

