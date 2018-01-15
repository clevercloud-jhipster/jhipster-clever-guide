# Deploying a Micro Service architecture on Clever Cloud

If you don't know CleverCloud, you should have a look ;o) It's a Cloud plateform blah blah blah...

The purpose of this post is to explain you how to deploy a microservice architecture on CleverCloud.
The business case of the application is pretty simple: persisting books into a relational database. 
But, when you create a new book, you need an ISBN number, and that's when microservices become handy.
Both Book and ISBN are separate microservices and need to talk to each other.
Also, like any real world application, we need authentication and configuration.

## Presentation

No matter how simple your business logic is, when you use a microservice architecture, you need a bit of presentation and some diagrams so people can understand. 

### Architecture

The overall architecture looks like this.
    
### Book API

The Book API just does CRUD operations on a Book entity (title, author and isbn). Nothing special except
when creating a new book: the REST controller uses [Feign](https://github.com/OpenFeign/feign) to communicate with the ISBN API to get back an isbn number.

    @PostMapping("/books")
    @Timed
    public ResponseEntity<Book> createBook(@RequestBody Book book) throws URISyntaxException {

        ResponseEntity<String> stringResponseEntity = isbnService.generateIsbnNumberUsingGET();
        String isbn = stringResponseEntity.getBody();

        book.setIsbn(isbn);

        Book result = bookRepository.save(book);
        return ResponseEntity.created(new URI("/api/books/" + result.getId()))
            .headers(HeaderUtil.createEntityCreationAlert(ENTITY_NAME, result.getId().toString()))
            .body(result);
    }

### ISBN API

The ISBN API is dead simple: it's just a REST endpoint that returns a random number:

    @GetMapping("/isbns/number")
    @Timed
    public String generateIsbnNumber() {
        log.debug("REST request to get generate an Isbn number");
        return "ISBN-" + String.valueOf(Math.random());
    }

### Gateway

In JHipster, the [gateway](http://www.jhipster.tech/api-gateway/) acts as the entrance to our microservices. 
It provides an Angular user interface with HTTP routing and load balancing, quality of service, security and API 
documentation for all microservices.

This is what the gateway looks like in our application.

### JHispter Registry

The [JHipster Registry](http://www.jhipster.tech/jhipster-registry/) has three main purposes:

* It is a an [Eureka server](https://cloud.spring.io/spring-cloud-netflix/spring-cloud-netflix.html), that serves as a discovery server for applications. This is how JHipster handles routing, load balancing and scalability for all applications.
* It is a [Spring Cloud Config server](https://cloud.spring.io/spring-cloud-config/spring-cloud-config.html), that provide runtime configuration to all applications.
* It is an administration server, with dashboards to monitor and manage applications.

In our architecture, the Registry is used to register and lookup the Gateway, Book and ISBN API.

### Keycloak

A real world application needs some soft of security. And when you are in the Cloud and that you expose APIs, it's better to have a good security support. JHipster proposes 
[different options](http://www.jhipster.tech/security/), and we choose [Keycloak](http://www.keycloak.org/).
With Keycloak, no need to deal with storing users or authenticating users, it secures our services with minimum fuss (and when I say services, I mean we secure our APIs as weel 
as our gateway). We can even get advanced features such as User Federation, Identity Brokering and Social Login.
In our use case we wanted to plug Keycloak to a real LDAP server ([Apache DS](http://directory.apache.org/)) 

## Building

### APIs and Gateway

To build our Book and ISBN APIs we rely on our good old Maven. JHipster comes with different profiles, [production](http://www.jhipster.tech/production/) is one of them. So it's
 just a matter of executing a `./mvnw -Pprod`.
 
It is the same for the Gateway... except that Maven uses the webpack plugin to build the front-end (`run webpack:prod`) 

## Deploying on Clever Cloud

### JHipster addon

## Scaling on Clever Cloud


