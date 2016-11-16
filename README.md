# Standardization proposal for the "French consultations portal" deployment API

## Context

This document is a proposed specification for the deployment APIs that consultation service providers will be required to develop for allowing the French consultations portal to implement a "one-click instant deploy".

## Terms & vocabulary

 * the "*platform*": the French consultations portal platform ;
 * the "*provider(s)*": the SaaS consultation provider service(s).

## Proposal

Allowing a smooth yet secure deployment requires a symetric set of APIs:

 * one part must be run and maintained by the service *provider* ([Cap Collectif](https://cap-collectif.com/) for instance, among others) ;
 * the other part must be run by the *platform*, to collect feedback and informations from the service providers.

This specification emphazises the "providers" part, but also gives some hints on the "*platform*" side:

 * [provider.swagger.yml](provider.swagger.yml) lists the minimal set of APIs that should be exposed by the *providers* ;
 * [platform.swagger.yml](platform.swagger.yml) lists the minimal set of APIs that should be exposed by the *platform*.

Some design principles:

 * These APIs communicate over http2 or https (with valid certs) ;
 * At first, API security will be handled through shared secrets (one secret per provider, no OAuth nor JWT). This approach should change after the first launch of the whole platform ;
 * Some solutions may require `spf` and `DKIM` dns entries to be created ;
 * it is strongly recommended that each solution will only be available to users through `https`, using an automated certification authority (eg. [Let's Encrypt](https://letsencrypt.org/) which is free and open). This, however, requires that DNS entries are created (and flushed) **before** the instance creation request is issued. Details in the chapter "Deployment Workflow" below ;

## Deployment Workflow

The basic deployment workflow includes several steps:

 * 1- a user requests a `example.consultation.gouv.fr` instance on the *platform* website. His request is (email) verified, rate-limited and, if it seems legitimate, the instance creation process is launched ;
 * 2- the *platform* creates the DNS entries, depending on the requirements of the *provider*:
   * a `CNAME` dns entry is created, which links `example.consultation.gouv.fr` to a DNS name given by the provider. **This supposes that all the consultation platforms created through this API will hit the very same front server - which is BAD for scalability. An improved version should allow the provider to change the CNAME target  for each instance through an API (as long as the default CNAME target)**
   * if required by the provider, `SPF` and `DKIM` entries have to be created for `@example.consultation.gouv.fr`. All the outgoing emails will be sent using a `no-reply@example.consultation.gouv.fr` email address.
 * 3- the provider's `/instances` creation endpoint is called by the *platform*, which then displays an informative message to the user ;
 * 4- the *provider* deploys the instance - this is done synchronously or asynchronously, and the *platform* will get an appropriate status code. Once completed, the deployment status and metadata are sent to the *platform*'s `/instances/{requestIdentifier}` endpoint using http's `PATCH` method. The "metadata" is a flat key-value array, which may contain several informations that the *provider* could find interesting to communicate to the *platform* (admin interface url, admin username, admin password, instance creation duration, http authentication credentials, etc.)

Additional instance-lifecycle management operations have not been documented using this workflow but could be useful in the future. Here are some ideas:

 * request the termination/archive of an instance (with a cancellation delay?) ;
 * force the immediate termination of an instance ;
 * get usage statistics (users count, contributions count) for deployed instances (or these endpoints should be exposed by the instances, not this deployment API?);

## Testing

The OpenAPI (Swagger) specification is an API description format which allows to describe an API in a non-ambiguous and very precise way.

You may use the [Swagger editor](http://editor.swagger.io/) to read the specification in a more graphical way:

 * [see the *providers* API definition](http://editor.swagger.io/#/?import=https://raw.githubusercontent.com/cap-collectif/consultation-gouv-fr-deployment-api/master/provider.swagger.yml) ;
 * [see the *platform*'s API definition](http://editor.swagger.io/#/?import=https://raw.githubusercontent.com/cap-collectif/consultation-gouv-fr-deployment-api/master/platform.swagger.yml) ;

You may generate implementations of both APIs clients in various languages by using the [Swagger codegen](https://github.com/swagger-api/swagger-codegen) tool.

## Contributing

Contributions to this document are welcome as pull-requests.

## License

This code/content is published under the MIT License.
