# Why ksonnet.next?

Writing a Kubernetes app from scratch can be tedious.

Even when you've gotten your core application logic packaged neatly away into a Docker image, you're not done yet! You still need to ensure that your JSON/YAML manifest files are written properly. Unless you've memorized the Kubernetes API, writing these manifests usually involves a cross-referencing spree against the API spec, code examples, online blogposts, and other eclectic sources.

It gets "better" with time, experience--and a growing history of previously-used manifest files that that you can copy.

Maybe this is not the best way.

~

Writing a web app from scratch used to be tedious.

Even when you'd written a "working" MVP with front-end code, server-side code, and accompanying database schemas, it was hard to ensure that all of these different pieces tied together in a readable, maintainable way. How could you prototype quickly in a way that didn't require massive refactoring later on?

(Maybe this was not the best way.)

~

Then along came web frameworks like Rails and Django.

With a few easy CLI commands (`rails new blog`), you can now autogenerate entire directory structures and skeleton code. Rather than having to enforce MVC (model-view-controller) yourself, you can rest easy that the "glue" of your app conforms to best practices. With less overhead, you develop faster, and can focus on the problem domain that you actually care about.

~

**ksonnet.next is meant to be for Kubernetes manifests what Ruby on Rails is for web apps.**

You can roughly think of it as the following combination:

```
Jsonnet + Kubernetes API spec + “kubectl” + “Rails”
```

* *Reusability and extensibility with **Jsonnet***

  Jsonnet is a flexible, JSON/YAML-compatible data templating language that allows you to better organize your configuration files with reusable components called mixins. As your application scales up, mixins simplify the work that's required to extend your configuration. They allow you to write your manifests according to a separation of concerns, such that your logging subteam's mixin might be reused across search and payment product subteams.

* *Validation against the **Kubernetes API***

  Ksonnet.next allows you to specify the exact version of the Kubernetes API spec that your app should be compatible with. It automatically provides the appropriate mixins for you to work with, as well as validation functionality.

* *Manage manifests for an actual cluster, **similar to kubectl***

  Similarly to kubectl, ksonnet.next allows you to apply, compare, and delete your local manifests to those on a remote cluster--regardless of whether your manifests are written in JSON, YAML, or Jsonnet.

* *Manage environments, mixins, and common templates with **a framework like Rails***

  Similarly to how Rails auto-generates the skeleton of a web app that follows the MVC pattern, ksonnet.next auto-generates the skeleton of a well-organized Kubernetes app. It provides structure that makes it easy to manage this app across environments (dev vs. prod, or the various availability zones of an HA cluster) and to reuse 3rd-party code.

~

Writing a Kubernetes app doesn't have to be tedious.

With its declarative framework, ksonnet.next simplifies the task of managing your Kubernetes applications and their associated configurations. Not only does it make it easy to get started, it also allows you to enforce the best practice of "configuration-as-code". You can use ksonnet.next to express common patterns across your infrastructure, reuse these powerful "templates" across many services, and manage those templates as files in version control. In fact, the more complex your infrastructure is, the more you will gain from using ksonnet.next.

To see an example of ksonnet.next being used in practice, see [the tutorial doc](/docs/tutorial.md).

> NOTES:
> * Figure out if this explanation is necessary and where it should go (website, blog post?)
> * Does it read too much like a powerpoint presentation?
> * Make sure things are accurate--no false advertising
> * Is the structure too in-your-face obvious
> * Condense
> * Figure out tenses
> * How commonly used is the term "manifests" as opposed to "configurations"?
