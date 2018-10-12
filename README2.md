- SAS administrator password pout-mores-vibrancy

- password for default user pout-mores-vibrancy

- connecting to bastion, then ansible controller, then other hosts.

- proxy config for development/testing/troubleshooting
-
        sudo tee /etc/profile.d/http_proxy.sh <<\EOF
        export http_proxy="http://proxy.cloudopsprod.aws.velocityfrequentflyer.internal:3128"
        export https_proxy=$http_proxy
        export HTTP_PROXY=$http_proxy
        export HTTPS_PROXY=$http_proxy
        export no_proxy=".internal,.sas,.viya.sas,localhost,169.254.169.254"
        export NO_PROXY=$no_proxy
EOF
        sudo chmod +x /etc/profile.d/http_proxy.sh
        . /etc/profile.d/http_proxy.sh

- proxy config in bash shell scripts

- proxy config in ansible playbooks

- mirror upload
        mirrormgr mirror --deployment-data ~/SAS_Viya_deployment_data.zip --platform x64-redhat-linux-6 --latest   --remove-old
        mirrormgr mirror --deployment-data ~/SAS_Viya_deployment_data.zip --platform x64-redhat-linux-7 --latest   --remove-old
        aws s3 sync $HOME/sas_repos/ s3://sas-viya-installation/mirror/
