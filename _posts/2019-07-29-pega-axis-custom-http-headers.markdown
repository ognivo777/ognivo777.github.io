---
layout: post
title:  "How-to add custom HTTP header to each SOAP request in Pega PRPC"
---
Open rule **ACTIVITY Rule-Connect-SOAP • InvokeAxis2 ** and add **java** step before the step with description like "*Invoke the remote web service*" with next content:
```java
//prepare some data for audit (not in scope) and used in headers
String workPageName = tools.getParamValue("WorkPageName");
    if (workPageName == null || workPageName.trim().length() == 0) {
        oLog.warn("WorkPageName parameter was not specified.");
        workPageName = "pyWorkPage";
    }

    ClipboardPage woPage = tools.findPage(workPageName);
    if (woPage != null) {
        if (oLog.isDebugEnabled()) {
            oLog.debug("Integration audit parameters will be taken from '" + woPage.getName() + "'");
        }
        tools.putParamValue("pAuditInsKey", woPage.getString(".pzInsKey"));
        tools.putParamValue("pAuditCoverInsKey", woPage.getString(".pxCoverInsKey"));
    }
    
    tools.putParamValue("pAuditServiceName", tools.getParamValue("ServiceName"));
    tools.putParamValue("pAuditRequestTime", ThreadContainer.get().getDateTimeUtils().getCurrentTimeStamp());
    
} catch (Exception ex) {
    oLog.error("Integration audit prepare procedure failed: " + ex, ex);
}

com.pega.apache.axis2.client.OperationClient client = (com.pega.apache.axis2.client.OperationClient) opClient;
com.pega.apache.axis2.client.Options options = client.getOptions();

String endpointName = client.getOperationContext().getAxisOperation().getAxisService().getEndpointName();
oLog.infoForced(".getEndpointName(): " + endpointName);

String internalSoapURL = options.getTo().getAddress();
oLog.infoForced(".getAddress(): " + internalSoapURL);
tools.getParameterPage().putString("pAuditServiceDestinationURL", internalSoapURL);

try{
  if(options==null) {
    oLog.infoForced("msgContext is null :(");
  } else {
    java.util.List headersTable = new java.util.ArrayList();

    headersTable.add(new com.pega.apache.commons.httpclient.Header("pAuditInsKey", tools.getParamValue("pAuditInsKey")));
    headersTable.add(new com.pega.apache.commons.httpclient.Header("pAuditCoverInsKey", tools.getParamValue("pAuditCoverInsKey")));
    headersTable.add(new com.pega.apache.commons.httpclient.Header("pAuditServiceName", tools.getParamValue("pAuditServiceName")));
    headersTable.add(new com.pega.apache.commons.httpclient.Header("pAuditRequestTime", tools.getParamValue("pAuditRequestTime")));

    options.setProperty(com.pega.apache.axis2.transport.http.HTTPConstants.HTTP_HEADERS, headersTable);
  }
} catch (Exception e) {
    oLog.error("Error on add test http header:", e);
}
```
