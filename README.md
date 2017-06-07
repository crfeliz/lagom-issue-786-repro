Rerpoduces [lagom issue 786](https://github.com/lagom/lagom/issues/786)

See [ed40651](https://github.com/crfeliz/lagom-issue-786-repro/commit/ed4065186e727adcc885e1f75482ef4091a16fa6) for changes made to the default lagom hello project. 

To reproduce
- sbt runAll
- curl the hello endpoint
```
curl -X POST \
  http://localhost:9000/api/hello/bug \
  -H 'cache-control: no-cache' \
  -H 'content-type: application/json' \
  -H 'postman-token: 6fcef9df-7bd0-04b3-f808-7bcdd586183f' \
  -d '{"name":"drumroll...", "message": "kaboom!"}'
```
- you should see the following warnings:
```
[warn] a.s.Serialization(akka://hello-impl-application) - Multiple serializers found for class com.example.hello.impl.GreetingMessageChanged, choosing first: Vector((interface java.io.Serializable,akka.serialization.JavaSerializer@7b9c3f10), (interface com.example.hello.impl.HelloEvent,com.lightbend.lagom.scaladsl.playjson.PlayJsonSerializer@6f2d0596))
[warn] a.s.Serialization(akka://hello-impl-application) - Using the default Java serializer for class [com.example.hello.impl.GreetingMessageChanged] which is not recommended because of performance implications. Use another serializer or disable this warning using the setting 'akka.actor.warn-about-java-serializer-usage'
```
- uncomment line 14 in the hello-impl project's application.conf (`"java.io.Serializable" = none`)
- curl it again (same request)
- you should see the following errors:
```
[error] hello - Exception in PathCallIdImpl(/api/hello/:id)
com.lightbend.lagom.scaladsl.persistence.PersistentEntity$PersistException: Persist of [akka.persistence.journal.Tagged] failed in [com.example.hello.impl.HelloEntity] with id [bug], caused by: {Missing play-json serializer for [com.example.hello.impl.GreetingMessageChanged]
[error] c.l.l.i.s.p.PersistentEntityActor - Failed to persist event type [akka.persistence.journal.Tagged] with sequence number [2] for persistenceId [HelloEntity|bug].
```
