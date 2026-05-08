ARG SOLR_VERSION=9.10.1
ARG DATAVERSE_VERSION=6.10.1

# Download schema.xml and solrconfig.xml from the matching IQSS Dataverse release.
# Using the release tarball keeps the conf version-locked to the Dataverse version.
FROM alpine:3 AS dataverse-solr-conf
ARG DATAVERSE_VERSION
RUN apk add --no-cache curl && \
    mkdir -p /solr-conf && \
    BASE="https://raw.githubusercontent.com/IQSS/dataverse/v${DATAVERSE_VERSION}/conf/solr" && \
    curl -fsSL "${BASE}/schema.xml"    -o /solr-conf/schema.xml && \
    curl -fsSL "${BASE}/solrconfig.xml" -o /solr-conf/solrconfig.xml

# Final image: start from the matching Solr release, copy in the _default configset
# (which provides lang/, stopwords.txt, synonyms.txt, etc.), then overlay the
# Dataverse-specific schema and solrconfig fetched above.
FROM solr:${SOLR_VERSION}
USER root
RUN mkdir -p /opt/solr/server/solr/configsets/dataverse && \
    cp -r /opt/solr/server/solr/configsets/_default/conf \
         /opt/solr/server/solr/configsets/dataverse/conf && \
    rm -f /opt/solr/server/solr/configsets/dataverse/conf/managed-schema.xml && \
    chown -R solr:solr /opt/solr/server/solr/configsets/dataverse
COPY --from=dataverse-solr-conf --chown=solr:solr /solr-conf/ \
    /opt/solr/server/solr/configsets/dataverse/conf/
USER solr

CMD ["solr-precreate", "dataverse", "/opt/solr/server/solr/configsets/dataverse"]

