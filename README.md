SCS ESI Sample
==============

[Deutsche Anleitung zum Starten des Beispiels](WIE-LAUFEN.md)

For [Self-contained Systems](http://scs-architecture.org) multiple
frontends need to be integrated. This sample shows how to do
this. Varnish serves as a cache and also ESI (Edge Side Includes) are
used to integrate multiple backends into one HTTP site.

This project creates a Docker setup with the complete systems based on
Docker Compose and Docker Machine. The services are implemented in
Java using Spring and Spring Cloud.

It uses two very simple SCS:
- Order to enter orders.
- Catalog to handle the items in the catalog.

How To Run
----------

See [How to run](HOW-TO-RUN.md) for details.

Remarks on the Code
-------------------

The SCS are: 
- [scs-demo-esi-common](scs-demo-esi-common) provides web assets and common headers
and footers for services. The microservice is written in Go and therfore has a small
footprint. It uses Multi Stage Docker containers to compile the Go program in one Docker
image and then copy over the result into a separate Docker image.
- [scs-demo-esi-order](scs-demo-esi-order) does order processing. It outputs
  `esi:include` in its HTML files to include the HTML snippets of
  `scs-demo-esi-common`. These are interpreted by the Varnish web
  cache. This microservices is written in Java. It has an Java main application
  in [src/test/java](https://github.com/ewolff/SCS-ESI/tree/master/scs-demo-esi-order/src/test/java/com/ewolff/microservice/order)
  to run the microservice stand alone with some test data. However, the layout
  of the web page will be incomplete because the ESI includes are not handled.

Varnish interprets the ESIs. The `default.vcl` in the directory
  `docker/varnish` includes the configuration to enable this. It
  defines two backends - one for each SCS. HTTP requests to `/catalog`
  are mapped to the catalog SCS while the default is the order
  SCS. The time to live for each item in the cache is 30s i.e. after
  30 seconds an entry in the cache in invalidated and the call goes to
  the backend. The grace is 15m: Even if the entry in the cache is
  invalid it will be served for 15 more minutes of the backend is not
  accessible. So if the backend crashes the cache can still provide
  some resilience.



Architecture Disclaimer
-------------------

This is a technology demo. The coupling between the two components in
this case is very tight - the integration is providing rather small
components that are embedded in specific pages. In a real world
architecture this should not be the case. Please refer to
http://scs-architecture.org/ to better understand the architecture
this should support.
