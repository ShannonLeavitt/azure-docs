---
title: Certificates bindings
description: Explain numerous topics related to certificates on an App Service Environment v2. Learn how certificate bindings work on the single-tenanted apps in an ASE.
author: madsd
ms.topic: overview
ms.date: 03/29/2022
ms.author: madsd
---

# Certificates and the App Service Environment v2

> [!IMPORTANT]
> This article is about App Service Environment v2, which is used with Isolated App Service plans. [App Service Environment v1 and v2 are retired as of 31 August 2024](https://aka.ms/postEOL/ASE). There's a new version of App Service Environment that is easier to use and runs on more powerful infrastructure. To learn more about the new version, start with the [Introduction to the App Service Environment](overview.md). If you're currently using App Service Environment v1, please follow the steps in [this article](upgrade-to-asev3.md) to migrate to the new version.
>
> As of 31 August 2024, [Service Level Agreement (SLA) and Service Credits](https://aka.ms/postEOL/ASE/SLA) no longer apply for App Service Environment v1 and v2 workloads that continue to be in production since they are retired products. Decommissioning of the App Service Environment v1 and v2 hardware has begun, and this may affect the availability and performance of your apps and data.
>
> You must complete migration to App Service Environment v3 immediately or your apps and resources may be deleted. We will attempt to auto-migrate any remaining App Service Environment v1 and v2 on a best-effort basis using the [in-place migration feature](migrate.md), but Microsoft makes no claim or guarantees about application availability after auto-migration. You may need to perform manual configuration to complete the migration and to optimize your App Service plan SKU choice to meet your needs. If auto-migration isn't feasible, your resources and associated app data will be deleted. We strongly urge you to act now to avoid either of these extreme scenarios.
>
> If you need additional time, we can offer a one-time 30-day grace period for you to complete your migration. For more information and to request this grace period, review the [grace period overview](./auto-migration.md#grace-period), and then go to [Azure portal](https://portal.azure.com) and visit the Migration blade for each of your App Service Environments.
>
> For the most up-to-date information on the App Service Environment v1/v2 retirement, see the [App Service Environment v1 and v2 retirement update](https://github.com/Azure/app-service-announcements/issues/469).
>

The App Service Environment(ASE) is a deployment of the Azure App Service that runs within your Azure Virtual Network(VNet). It can be deployed with an internet accessible application endpoint or an application endpoint that is in your virtual network. If you deploy the ASE with an internet accessible endpoint, that deployment is called an External ASE. If you deploy the ASE with an endpoint in your virtual network, that deployment is called an ILB ASE. You can learn more about the ILB ASE from the [Create and use an ILB ASE](./create-ilb-ase.md) document.

The ASE is a single tenant system. Because it's single tenant, there are some features available only with an ASE that aren't available in the multitenant App Service.

## ILB ASE certificates

If you're using an External ASE, then your apps are reached at &lt;appname&gt;.&lt;asename&gt;.p.azurewebsites.net. By default all ASEs, even ILB ASEs, are created with certificates that follow that format. When you have an ILB ASE, the apps are reached based on the domain name that you specify when creating the ILB ASE. In order for the apps to support TLS, you need to upload certificates. Obtain a valid TLS/SSL certificate by using internal certificate authorities, purchasing a certificate from an external issuer, or using a self-signed certificate.

There are two options for configuring certificates with your ILB ASE.  You can set a wildcard default certificate for the ILB ASE or set certificates on the individual web apps in the ASE.  Regardless of the choice you make, the following certificate attributes must be configured properly:

- **Subject:** This attribute must be set to *.[your-root-domain-here] for a wildcard ILB ASE certificate. If creating the certificate for your app, then it should be [appname].[your-root-domain-here]
- **Subject Alternative Name:** This attribute must include both *.[your-root-domain-here] and*.scm.[your-root-domain-here] for the wildcard ILB ASE certificate. If creating the certificate for your app, then it should be [appname].[your-root-domain-here] and [appname].scm.[your-root-domain-here].

As a third variant, you can create an ILB ASE certificate that includes all of your individual app names in the SAN of the certificate instead of using a wildcard reference. The problem with this method is that you need to know up front the names of the apps that you're putting in the ASE or you need to keep updating the ILB ASE certificate.

### Upload certificate to ILB ASE

After an ILB ASE is created in the portal, the certificate must be set for the ILB ASE. Until the certificate is set, the ASE will show a banner that the certificate wasn't set.  

The certificate that you upload must be a .pfx file. After the certificate is uploaded, there's a time delay of approximately 20 minutes before the certificate is used.

You can't create the ASE and upload the certificate as one action in the portal or even in one template. As a separate action, you can upload the certificate using a template as described in the [Create an ASE from a template](./create-from-template.md) document.  

If you want to create a self signed certificate quickly for testing, you can use the following bit of PowerShell:

```azurepowershell-interactive
$certificate = New-SelfSignedCertificate -certstorelocation cert:\localmachine\my -dnsname "*.internal.contoso.com","*.scm.internal.contoso.com"

$certThumbprint = "cert:\localMachine\my\" + $certificate.Thumbprint
$password = ConvertTo-SecureString -String "CHANGETHISPASSWORD" -Force -AsPlainText

$fileName = "exportedcert.pfx"
Export-PfxCertificate -cert $certThumbprint -FilePath $fileName -Password $password
```

When creating a self signed cert, you'll need to ensure the subject name has the format of CN={ASE_NAME_HERE}_InternalLoadBalancingASE.

## Application certificates

Apps that are hosted in an ASE can use the app-centric certificate features that are available in the multitenant App Service. Those features include:  

- SNI certificates
- IP-based SSL, which is only supported with an External ASE.  An ILB ASE doesn't support IP-based SSL.
- KeyVault hosted certificates

The instructions for uploading and managing those certificates are available in [Add a TLS/SSL certificate in Azure App Service](../configure-ssl-certificate.md).  If you're simply configuring certificates to match a custom domain name that you have assigned to your web app, then those instructions will suffice. If you're uploading the certificate for an ILB ASE web app with the default domain name, then specify the scm site in the SAN of the certificate as noted earlier.

## TLS settings

You can configure the TLS setting at an app level.  

## Private client certificate

A common use case is to configure your app as a client in a client-server model. If you secure your server with a private CA certificate, you will need to upload the client certificate to your app.  The following instructions will load certificates to the truststore of the workers that your app is running on. If you load the certificate to one app, you can use it with your other apps in the same App Service plan without uploading the certificate again.

To upload the certificate to your app in your ASE:

1. Generate a *.cer* file for your certificate.
2. Go to the app that needs the certificate in the Azure portal
3. Go to SSL settings in the app. Click Upload Certificate. Select Public. Select Local Machine. Provide a name. Browse and select your *.cer* file. Select upload.
4. Copy the thumbprint.
5. Go to Application Settings. Create an App Setting WEBSITE_LOAD_ROOT_CERTIFICATES with the thumbprint as the value. If you have multiple certificates, you can put them in the same setting separated by commas and no whitespace like

 84EC242A4EC7957817B8E48913E50953552DAFA6,6A5C65DC9247F762FE17BF8D4906E04FE6B31819

The certificate will be available by all the apps in the same app service plan as the app, which configured that setting. If you need it to be available for apps in a different App Service plan, you'll need to repeat the App Setting operation in an app in that App Service plan. To check that the certificate is set, go to the Kudu console and issue the following command in the PowerShell debug console:

```azurepowershell-interactive
dir cert:\localmachine\root
```

To perform testing, you can create a self signed certificate and generate a *.cer* file with the following PowerShell:

```azurepowershell-interactive
$certificate = New-SelfSignedCertificate -certstorelocation cert:\localmachine\my -dnsname "*.internal.contoso.com","*.scm.internal.contoso.com"

$certThumbprint = "cert:\localMachine\my\" + $certificate.Thumbprint
$password = ConvertTo-SecureString -String "CHANGETHISPASSWORD" -Force -AsPlainText

$fileName = "exportedcert.cer"
export-certificate -Cert $certThumbprint -FilePath $fileName -Type CERT
```
