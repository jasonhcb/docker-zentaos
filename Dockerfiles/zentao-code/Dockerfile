FROM registry.saas.hand-china.com/rdchip/zentaopms-env:latest

MAINTAINER <chubo.he@hand-china.com>

COPY conf/nginx-site.conf /etc/nginx/sites-enabled/default.conf

RUN mkdir /app/src/public/zentaopms
ADD ./zentaopms /app/src/public/zentaopms
RUN curl -O http://files.saas.hand-china.com/hip/rdc/zentaopms.tar 
RUN tar -zxf zentaopms.tar -C /app/src/public/zentaopms
RUN rm zentaopms.tar
# ADD http://files.saas.hand-china.com/hip/rdc/zentaopms.tar /app/src/public/zentaopms/
#RUN ls -lh /app/src/public/zentaopms/
#RUN cd /app/src/public/zentaopms/ \
#	&& tar -zxf zentaopms.tar -C /app/src/public/zentaopms/
#RUN ls -lh /app/src/public/zentaopms/
#RUN rm /app/src/public/zentaopms/zentaopms.tar 
# COPY conf/my.php /app/src/public/zentaopms/config/my.php
RUN chown -R www-data:www-data /app/src/public/zentaopms/tmp/
RUN mkdir /app/src/public/zentaopms/module/hdc/data -p


RUN chmod -R 777 /app/src/public/zentaopms/www
#COPY conf.sh /conf.sh

# VOLUME  ["/app/src/public/zentaopms"]

#CMD /conf.sh