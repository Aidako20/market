#!/bin/bash
sudo adduser --system --quiet --shell=/bin/bash --home=/opt/market --gecos 'market' --group market
sudo mkdir /etc/market && mkdir /var/log/market/
sudo apt-get update && apt-get upgrade -y && apt-get install postgresql postgresql-server-dev-14 build-essential python3-pillow python3-lxml python3-dev python3-pip python3-setuptools npm nodejs git gdebi libldap2-dev libpq-dev libsasl2-dev libxml2-dev libxslt1-dev libjpeg-dev -y
sudo pip3 install --upgrade pip
sudo service postgresql restart
git clone --depth=1 --branch=3.0 https://github.com/Aidako20/market.git /opt/market/market
sudo chown market:market /opt/market/ -R && sudo chown market:market /var/log/market/ -R && cd /opt/market/market && sudo pip3 install -r requirements.txt
sudo npm install -g less less-plugin-clean-css rtlcss -y
cd /tmp && wget https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-3/wkhtmltox_0.12.6.1-3.jammy_amd64.deb && sudo gdebi -n wkhtmltox_0.12.6.1-3.jammy_amd64.deb && rm wkhtmltox_0.12.6.1-3.jammy_amd64.deb
sudo ln -s /usr/local/bin/wkhtmltopdf /usr/bin/ && sudo ln -s /usr/local/bin/wkhtmltoimage /usr/bin/
sudo su - postgres -c "createuser -s market"
sudo su - market -c "/opt/market/market/market-bin --addons-path=/opt/market/market/addons -s --stop-after-init"
sudo mv /opt/market/.marketrc /etc/market/market.conf
sudo sed -i "s,^\(logfile = \).*,\1"/var/log/market/market-server.log"," /etc/market/market.conf
sudo sed -i "s,^\(logrotate = \).*,\1"True"," /etc/market/market.conf
sudo sed -i "s,^\(proxy_mode = \).*,\1"True"," /etc/market/market.conf
sudo cp /opt/market/market/debian/init /etc/init.d/market && chmod +x /etc/init.d/market
sudo ln -s /opt/market/market/market-bin /usr/bin/market
sudo update-rc.d -f market start 20 2 3 4 5 .
sudo service market restart
