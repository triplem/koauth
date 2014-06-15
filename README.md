# KOauth - Asynchronous Scala Library for OAuth 1.0a

This library aids calculations according to the [Oauth 1.0a](http://oauth.net/core/1.0a/)
specifications for both HTTP server and client.

* Provider library: Verifying and responding to HTTP requests according to OAuth 1.0 specifications.
* Consumer library: Complementing HTTP requests to be sent with OAuth parameters. 

## Set up your project dependencies

If you're using SBT, clone this project, build and publish it to your local repository and
add to your project as a dependency.

Building and publishing locally:

```
git clone https://github.com/kovacshuni/koauth.git
cd koauth
sbt
compile
publish-local
```

I also don't understand yet all these versions of Scala and SBT and numbers concatenated to
artifact names with underscores and the single/double percent stuff, so i just hardcoded 2.11.

Add as dependency by editing your `build.sbt` and adding the following:

```scala
libraryDependencies ++= Seq(
  "com.hunorkovacs" % "koauth_2.11" % "1.0"
)
```

Or if you want to be on the bleeding edge using snapshots:

```scala
libraryDependencies ++= Seq(
  "com.hunorkovacs" % "koauth_2.11" % "1.0-SNAPSHOT"
)
```

koauth is available on Maven Central. No it's not. Not yet. But i'll try to publish it soon.

```scala
resolvers += "Sonatype Releases" at "https://oss.sonatype.org/content/repositories/releases/"
```

or 

```scala
resolvers += "Sonatype Releases" at "https://oss.sonatype.org/content/repositories/snapshots/"
```

## Step By Step Example - Provider

How to integrate OAuth verification in your REST API server, using this library.

There is also an sample project: [koauth-sample-play](https://github.com/kovacshuni/koauth-sample-play)
that you can most easily follow. It is using [Play Framework](http://www.playframework.com/) 2.2.2
as web framework, [MongoDB](http://www.mongodb.org/) with [ReactiveMongo](http://reactivemongo.org/)
to save client credentials and other details. Worth to use as the starting point, because it is
easier to understand.

### Define the HTTP paths required by OAuth 1

* POST to /oauth/request-token
* POST to /oauth/authorize
* POST to /oauth/access-token

Your exact paths may differ, but for a proper OAuth 1 you'll need to define at least 3 for these above.
You will probably want paths, to register a new user, verify their email, read the rights a token gives
access to, and invalidate tokens but that not mandatory here, but your app's part to do.

### Mapping your HTTP requests and responses.

You will need to implement the `OauthRequestMapper` and `OauthResponseMapper` to map your web
framework's HTTP request and response types to the library's independent types.
After that, you can easily call the service methods of the koauth library.
You may also want to catch the exception that may occur during authentication and return your specific
HTTP responses accordingly.

```scala
object PlayToOauthRequestMapper extends OauthRequestMapper[Request[AnyContent]] {
  override def map(source: Request[AnyContent])(implicit ec: ExecutionContext): Future[OauthRequest] = {
    Future {
      OauthRequest(source.headers("Authorization"),
        "http://" + source.host.toLowerCase + "/" + source.path,
        source.method.toUpperCase,
        source.queryString)
    }
  }
}

object OauthToPlayResponseMapper extends OauthResponseMapper[SimpleResult] {
  override def map(source: Future[OauthResponseOk])(implicit ec: ExecutionContext): Future[SimpleResult] = {
    source.map(r => Ok(r.body))
  }
}

object OauthController extends Controller {
  /**
   * Mapped to POST /oauth/request-token
   */
  def requestToken = Action.async { request =>
    try {
      val oauthRequestF = PlayToOauthRequestMapper.map(request)

      val oauthResponseF = OauthService.requestToken(oauthRequestF)

      OauthToPlayResponseMapper.map(oauthResponseF)
    } catch {
      case badRequestEx: OauthBadRequestException => Future(BadRequest(badRequestEx.message))
      case unauthorizedEx: OauthUnauthorizedException => Future(Unauthorized(unauthorizedEx.message))
    }
  }
}
```

## Step By Step Example - Consumer

How to build HTTP requests that should be OAuth signed, using this library.

todo write this readme part

## Owner

Hunor Kovács,
kovacshuni@yahoo.com
[hunorkovacs.com](http://www.hunorkovacs.com)

## Licence

Licensed under the [Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0) .
