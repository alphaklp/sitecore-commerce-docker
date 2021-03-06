# escape=`

# Stage 0: prepare files
FROM microsoft/aspnet:4.7.1-windowsservercore-1709 AS prepare

ARG COMMERCE_SIF_PACKAGE
ARG COMMERCE_CONNECT_PACKAGE
ARG COMMERCE_MA_PACKAGE
ARG COMMERCE_MA_FOR_AUTOMATION_ENGINE_PACKAGE
ARG COMMERCE_XPROFILES_PACKAGE
ARG COMMERCE_XANALYTICS_PACKAGE

SHELL ["powershell", "-NoProfile", "-Command", "$ErrorActionPreference = 'Stop';"]

ADD files/ /Files/

RUN Expand-Archive -Path "/Files/$Env:COMMERCE_SIF_PACKAGE" -DestinationPath /Files/CommerceSIF -Force; `
    Expand-Archive -Path "/Files/$Env:COMMERCE_CONNECT_PACKAGE" -DestinationPath /Files/SitecoreCommerceConnectCore -Force; `
    Expand-Archive -Path "/Files/$Env:COMMERCE_MA_PACKAGE" -DestinationPath /Files/CommerceMACore -Force; `
    Expand-Archive -Path "/Files/$Env:COMMERCE_MA_FOR_AUTOMATION_ENGINE_PACKAGE" -DestinationPath /Files/CommerceMACoreForAE -Force; `
    Expand-Archive -Path "/Files/$Env:COMMERCE_XPROFILES_PACKAGE" -DestinationPath /Files/CommerceXProfiles -Force; `
    Expand-Archive -Path "/Files/$Env:COMMERCE_XANALYTICS_PACKAGE" -DestinationPath /Files/CommerceXAnalytics -Force

# Stage 1: perform actual build
FROM sitecore-docker_sitecore

SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

ARG COMMERCE_CERT_PATH
ARG ROOT_CERT_PATH
ARG WEB_TRANSFORM_TOOL
ARG COMMERCE_CONNECT_ENGINE_PACKAGE

COPY --from=prepare /Files/CommerceSIF /Files/CommerceSIF/
COPY --from=prepare /Files/SitecoreCommerceConnectCore /Files/SitecoreCommerceConnectCore
COPY --from=prepare /Files/CommerceMACore /Files/CommerceMACore
COPY --from=prepare /Files/CommerceMACoreForAE /Files/CommerceMACoreForAE
COPY --from=prepare /Files/CommerceXProfiles /Files/CommerceXProfiles
COPY --from=prepare /Files/CommerceXAnalytics /Files/CommerceXAnalytics
COPY --from=prepare /Files/${WEB_TRANSFORM_TOOL} /Files/Microsoft.Web.XmlTransform.dll
COPY --from=prepare /Files/${COMMERCE_CONNECT_ENGINE_PACKAGE} /Files/Sitecore.Commerce.Engine.Connect.update
COPY --from=prepare /Files/${ROOT_CERT_PATH} /Files/${ROOT_CERT_PATH}

ADD sitecore/InstallCommercePackages.ps1 /Scripts/
ADD files/$COMMERCE_CERT_PATH /Files/
ADD scripts/Import-Certificate.ps1 /Scripts/

# Trust Self signed certificates
RUN /Scripts/Import-Certificate.ps1 -certificateFile /Files/$Env:COMMERCE_CERT_PATH -secret 'secret' -storeName 'Root' -storeLocation 'LocalMachine'; `
    /Scripts/Import-Certificate.ps1 -certificateFile /Files/$Env:ROOT_CERT_PATH -secret 'secret' -storeName 'Root' -storeLocation 'LocalMachine'

# Import XConnect certificate
RUN /Scripts/Import-Certificate.ps1 -certificateFile /Files/$Env:COMMERCE_CERT_PATH -secret 'secret' -storeName 'My' -storeLocation 'LocalMachine'; `
    /Scripts/Import-Certificate.ps1 -certificateFile /Files/$Env:ROOT_CERT_PATH -secret 'secret' -storeName 'My' -storeLocation 'LocalMachine'

# Patch Solr config, set initializeOnAdd = true
# See: https://sitecore.stackexchange.com/questions/11523/index-sitecore-marketingdefinitions-master-was-not-found-exception-in-sitecore
ADD sitecore/Sitecore.Commerce.Engine.Connectors.Index.Solr.InitializeOnAdd.config /Files/
RUN cp /Files/Sitecore.Commerce.Engine.Connectors.Index.Solr.InitializeOnAdd.config /inetpub/wwwroot/sitecore/App_Config/Include/zSitecore.Commerce.Engine.Connectors.Index.Solr.InitializeOnAdd.config

ENTRYPOINT /Scripts/Watch-Directory.ps1 -Path C:\Workspace -Destination c:\inetpub\wwwroot\sitecore
