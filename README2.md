- RAID 0 not implemented, because if the instance is stopped,started the ephemeral storage is wiped, along with the RAID 0 configuration
- placement groups not implemented because AWS recommends all instances to be of the same instance FAMILY (e.g. r4)
- SAS administrator password pout-mores-vibrancy
- password for default user pout-mores-vibrancy
- connecting to bastion, then ansible controller, then other hosts.

cat > /etc/profile.d/http_proxy.sh <<\EOF
export http_proxy="http://proxy.cloudopsprod.aws.velocityfrequentflyer.internal:3128"
export https_proxy=$http_proxy
export HTTP_PROXY=$http_proxy
export HTTPS_PROXY=$http_proxy
export no_proxy=".internal,localhost,169.254.169.254"
export NO_PROXY=$no_proxy
EOF
chmod +x /etc/profile.d/http_proxy.sh
. /etc/profile.d/http_proxy.sh
