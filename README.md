# Standardization proposal for the OGP toolbox consultations deployment API

## Context

This document is a proposed specification for the deployment APIs that consultation service providers will be required to develop for allowing the OGP toolbox to implement a "one-click instant deploy".

## Terms & vocabulary

 * the "*platform*": the OGP Toolbox platform ;
 * the "*provider(s)*": the SaaS consultation provider service.

## Proposal

Allowing a smooth yet secure deployment requires a symetric set of APIs:

 * one part must be run and maintained by the service provider ([Cap Collectif](https://cap-collectif.com/) for instance, among others) ;
 * the other part must be run by Etalab/OGP, to collect feedback and informations from the service providers.

This specification emphazises the "providers" part, but also gives some hints on the "OGP" side:

 * [provider.swagger.yml](provider.swagger.yml) lists the set of APIs that should be exposed by the providers. Some of them are quite work-demanding and we cannot therefore expect all this work to be done by the service providers by Decembre 8th, 2016. The *MVP* APIs, required before December 8th, are tagged with the `MVP` tag ;
 * [platform.swagger.yml](platform.swagger.yml) lists the minimal APIs to be exposed by the OGP platform before December 8th, 2016.

Some design principles:

 * These APIs communicate over https (with valid certs) ;
 * At first, API security will be handled through shared secrets (one secret per provider, no OAuth or JWT). This approach could change after the first launch of the whole platform ;
 * Some solutions may require `spf` and `DKIM` dns entries to be created ;
 * it is strongly recommended that each solution will only be available to users through `https`, using an automated certification authority (eg. [Let's Encrypt](https://letsencrypt.org/) which is free and open). This, however, requires that DNS entries are created (and flushed) **before** the instance creation request is issued. Details in the chapter "Deployment Workflow" below ;

## Deployment Workflow

The basic deployment workflow includes several steps:

 * 1- a user requests a `example.consultation.gouv.fr` instance on the OGP toolbox website. His request is (email) verified, rate-limited and, if it seems legitimate, the instance creation process is launched ;
 * 2- the *platform* creates the DNS entries, depending on the requirements of the *provider*:
   * a `CNAME` dns entry is created, which links `example.consultation.gouv.fr` to a DNS name given by the provider. **This supposes that all the consultation platforms created through this API will hit the very same front server - which is BAD for scalability. An improved version should allow the provider to change the CNAME target  for each instance through an API (as long as the default CNAME target)**
   * if required by the provider, `SPF` and `DKIM` entries have to be created for `@example.consultation.gouv.fr`. All the outgoing emails will be sent using a `no-reply@example.consultation.gouv.fr` email address.
 * 3- the provider's `/instances` creation endpoint is called by the *platform*, which then displays an informative message to the user ;
 * 4- the *provider* deploys the instance - this is done asynchronously. Once completed, the deployment status and metadata are sent to the *platform*'s `/instances/{instanceId}` endpoint using http's `PATCH` method. The "metadata" is a flat key-value array, which may contain several informations that the *provider* could find interesting to communicate to the *platform* (admin interface url, admin username, admin password, instance creation duration, http authentication credentials, etc.)

Additional instance-lifecycle management operations have not been documented using this workflow. Here there are:

 * request the termination of an instance (already documented in the swagger files) ;
 * force the immediate termination of an instance ;
 * get usage statistics (users count, contributions count) for deployed instances ;

## Testing

The OpenAPI (Swagger) specification is an API description format which allows to describe an API in a non-ambiguous and very precise way.

You may use the [Swagger editor](http://editor.swagger.io/) to read the specification in a more graphical way:

 * [see the *providers* API definition](http://editor.swagger.io/#/?import=https://raw.githubusercontent.com/cap-collectif/consultation-gouv-fr-deployment-api/master/provider.swagger.yml) ;
 * [see the *platform*'s API definition](http://editor.swagger.io/#/?import=https://raw.githubusercontent.com/cap-collectif/consultation-gouv-fr-deployment-api/master/platform.swagger.yml) ;

## Provider referencing requirements

In order to reference a *provider*, the *platform* requires several informations:

 * `api endpoint`: API domain and path ;
 * does the *provider* support custom email-senders? (à la "XYZ@example.consultation.gouv.fr") If yes:
   * SPF: the OGP toolbox could either completely use the provider's SPF entries (`v=spf1 redirect=sample.cap-collectif.com`) or choose to include its rules (`v=spf1 include:sample.cap-collectif.com -all`). The latter could allow more flexibility to the *platform* but has some downsides ;
   * DKIM: this entry can only be added using informations given by the *provider* once the initial deployment has been done (the *provider* may use third-party mailing services). There is no good solution for this, except maybe allowing the *providers* to change their SPF and DKIM entries through a platform-exposed API.
 * what must be the `CNAME` target for the test-domains used for this *provider*?

## Questions / topics to tacle / @TODO

 * how to handle DNS load balancing accross several CNAMEs?
 * error samples
 * logging and auditing?

## Contributing

Contributions to this document are welcome as pull-requests.

## License

This code/content is published under the MIT License.
