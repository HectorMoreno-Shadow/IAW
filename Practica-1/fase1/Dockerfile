FROM httpd:2.4

COPY ./vhosts/00-default.conf /usr/local/apache2/conf/00-default.conf
COPY ./vhosts/marca1.conf /usr/local/apache2/conf/marca1.conf
COPY ./vhosts/marca2.conf /usr/local/apache2/conf/marca2.conf

RUN echo "Include conf/00-default.conf" >> /usr/local/apache2/conf/httpd.conf
RUN echo "Include conf/marca1.conf" >> /usr/local/apache2/conf/httpd.conf
RUN echo "Include conf/marca2.conf" >> /usr/local/apache2/conf/httpd.conf

EXPOSE 80

CMD ["httpd", "-D", "FOREGROUND"]