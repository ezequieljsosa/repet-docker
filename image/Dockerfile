FROM robsyme/docker-sge:latest

RUN apt update && apt install -y ncbi-blast+ blast2 gawk r-base \
  python-pip libmysqlclient-dev wget nano libfontconfig1-dev && \
  pip install MySQL-python pyyaml && \
  (curl -L http://cpanmin.us | perl - App::cpanminus) && cpanm Text::Soundex Bio::Perl

ADD programs /programs
WORKDIR /programs

#RUN tar xfv RepeatScout-1.0.5.tar.gz && cd RepeatScout-1 && make && make install && cd /programs \

RUN tar xfv REPET_linux-x64-2.5.tar.gz && mv REPET_linux-x64-2.5 /repet && cd /programs && \
 tar xfv genometools-1.5.9.tar.gz && cd genometools-1.5.9 && \
  make 64bit=yes  cairo=no && make install 64bit=yes  cairo=no && cd /programs && \
 tar xfv RECON-1.07.tar.gz && cd RECON-1.07/src && make && make install && \
 sed -i 's/$path = "";/$path = "\/programs\/RECON-1.07\/bin";/g' /programs/RECON-1.07/scripts/recon.pl && cd /programs && \
 mkdir piler && mv piler_source.tar.gz piler && cd piler && \
  tar xfv piler_source.tar.gz && make && cp /programs/piler/piler2 /usr/bin/piler && cd /programs && \
 tar xfv mcl-latest.tar.gz && cd mcl-14-137/ && ./configure && make && make install && cd /programs && \
 mv trf409.linux64 /usr/local/bin/trf && chmod +x /usr/local/bin/trf && cd /programs && \
 tar xfv mreps-2.6.tar && cd mreps && make && cp mreps /usr/bin/ && cd /programs && \
 tar xfv hmmer-3.2.1.tar.gz && cd hmmer-3.2.1 && ./configure && \
  make && make install && cd easel && make install && cd /programs && \
 tar xfv RepeatMasker-open-4-0-8.tar.gz && cd /programs && \
 sed -i 's/lArgs.append("-dir . -pa 1 -gccalc -no_is -int -s")/lArgs.append("-dir . -pa 1 -gccalc -no_is -noint -s")/' /repet/bin/TEannot.py && \
 dpkg -i libcppunit-1.12-1_1.12.1-4_amd64.deb && \
 ln -s /usr/lib/libcppunit-1.12.so.1 /usr/lib/libcppunit-1.12.so.0  && \
  /sbin/ldconfig -v && \
 sed -i "s/my ( $na, $tag2, $dfamLibraryVersion ) =//" /programs/RepeatMasker/configure && \
 sed -i 's/LibraryUtils::validateLibraries( "$rmLocation\/Libraries", "HMM" );//' /programs/RepeatMasker/configure && \
 tar zxvf ncbi-blast-2.6.0+-src.tar.gz && gunzip isb-2.6.0+-changes-vers2.patch.gz && \
    cd ncbi-blast-2.6.0+-src && patch -p1 < ../isb-2.6.0+-changes-vers2.patch && \
    cd c++ && ./configure --with-mt --without-debug && \
    make && cp /programs/ncbi-blast-2.6.0+-src/c++/ReleaseMT/bin/rmblastn /usr/bin && \
     cd /programs && rm mreps-2.6.tar && rm *.gz && rm -r mcl-14-137/ && rm -r genometools-1.5.9 && \
     rm -r hmmer-3.2.1 && rm -r mreps && rm -r /programs/RECON-1.07/Demos && \
     rm -r /programs/RECON-1.07/src && rm /programs/RepeatMasker/Libraries/Dfam* && \
     rm -r /programs/ncbi-blast-2.6.0+-src/ && \
 chown -R sgeuser:sgeuser /repet /programs
RUN chmod +x /usr/local/bin/trf
ADD *.cfg /docker_cfg/

RUN sed  -i 's|chr", "varchar(255|chr", "varchar(50|g' /repet/commons/core/sql/DbMySql.py
RUN sed  -i 's|name", "varchar(255|name", "varchar(50|g' /repet/commons/core/sql/DbMySql.py
RUN sed  -i 's|accession", "varchar(255|accession", "varchar(100|g' /repet/commons/core/sql/DbMySql.py
RUN sed  -i 's|description", "varchar(255|description", "varchar(100|g' /repet/commons/core/sql/DbMySql.py
RUN sed  -i 's|jobname", "varchar(255|jobname", "varchar(100|g' /repet/commons/core/sql/DbMySql.py


ENV PATH $PATH:/repet/bin/:/opt/sge/bin/lx-amd64/:/programs/RECON-1.07/scripts:/out/RepeatMasker:/out/censor_bin/bin

ENV PYTHONPATH $PYTHONPATH:/repet/
ENV SGE_ROOT /opt/sge
ENV USER sgeuser

ENV REPET_PATH /repet/
ENV REPET_JOB_MANAGER SGE
ENV REPET_HOST=mysql
ENV REPET_USER=root
ENV REPET_PW=repet
ENV REPET_DB=repet
ENV REPET_PORT=3306

WORKDIR /out
