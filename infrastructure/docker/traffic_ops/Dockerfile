############################################################
# Dockerfile to build Traffic Ops 1.6.0 container images
# Based on CentOS 6.6
############################################################

# Example Build and Run:
# docker network create cdnet
# docker build --rm --tag traffic_ops:1.6.0 --build-arg=RPM=http://traffic-control-cdn.net/downloads/1.6.0/RELEASE-1.6.0/traffic_ops-1.6.0-3503.4899d302.x86_64.rpm traffic_ops
#
# docker run --name my-traffic-ops-mysql --hostname my-traffic-ops-mysql --net cdnet --env MYSQL_ROOT_PASSWORD=secretrootpass --detach mysql:5.5
#
# docker run --name my-traffic-ops --hostname my-traffic-ops --net cdnet --publish 443:443 --env MYSQL_IP=my-traffic-ops-mysql --env MYSQL_PORT=3306 --env MYSQL_ROOT_PASS=secretrootpass --env MYSQL_TRAFFIC_OPS_PASS=supersecretpassword --env ADMIN_USER=superroot --env ADMIN_PASS=supersecreterpassward --env CERT_COUNTRY=US --env CERT_STATE=Colorado --env CERT_CITY=Denver --env CERT_COMPANY=NotComcast --env TRAFFIC_VAULT_PASS=marginallylesssecret --env DOMAIN=cdnet --detach traffic_ops:1.5.1

FROM centos:6.6
MAINTAINER Dan Kirkwood

RUN yum install -y tar gcc-c++ perl cpan openssl-devel mysql-devel expect

# Configure cpan to follow prereqs
RUN perl -MCPAN -e 'my $c = "CPAN::HandleConfig"; $c->load(doit => 1, autoconfig => 1); $c->edit(prerequisites_policy => "follow"); $c->edit(build_requires_install_policy => "yes"); $c->commit'

RUN cpan Carton

# Override RPM arg to use a different one using --build-arg RPM=...  Can be local file or http://...
ARG RPM=http://traffic-control-cdn.net/downloads/1.6.0/RELEASE-1.6.0/traffic_ops-1.6.0-3503.4899d302.x86_64.rpm
ADD $RPM /
RUN yum install -y /$(basename $RPM)

# once installed, remove rpm to lower image size
RUN rm /$(basename $RPM)

RUN cd /opt/traffic_ops/app && carton

# TODO: workaround for installing on CentOS 7 -- fix this!
RUN chmod a+x /opt/traffic_ops/app/bin/* \
	 /opt/traffic_ops/app/script/* \
	 /opt/traffic_ops/app/db/*.pl \
	 /opt/traffic_ops/app/db/*.sh \
	 /opt/traffic_ops/install/bin/*

RUN export PERL5LIB=/opt/traffic_ops/app/local/lib/perl5/:/opt/traffic_ops/install/lib/ \
	&& export TERM=xterm \
	&& export USER=root \
	&& yes | /opt/traffic_ops/install/bin/build_trafficops_perl_library -i

RUN export PERL5LIB=/opt/traffic_ops/app/local/lib/perl5/:/opt/traffic_ops/install/lib/ \
	&& export TERM=xterm \
	&& export USER=root \
	&& /opt/traffic_ops/install/bin/download_web_deps -i

# \todo add Drive Letters to postinstall input
RUN sed -i -- 's/"value": "b,c,d,e,f,g,h,i,j,k,l,m,n,o,p,q,r,s,t,u,v,w,x,y"/"value": "0"/g' /opt/traffic_ops/install/data/profiles/profile.trafficserver_edge.traffic_ops
RUN sed -i -- 's/"value": "b,c,d,e,f,g,h,i,j,k,l,m,n,o,p,q,r,s,t,u,v,w,x,y"/"value": "0"/g' /opt/traffic_ops/install/data/profiles/profile.trafficserver_mid.traffic_ops

RUN echo "{\"user\": \"riakuser\",\"password\": \"$TRAFFIC_VAULT_PASS\"}" > /opt/traffic_ops/app/conf/production/riak.conf

EXPOSE 443
WORKDIR /opt/traffic_ops/app
ENV MOJO_MODE production
ADD profile.origin.traffic_ops /
ADD run.sh /
CMD /run.sh
