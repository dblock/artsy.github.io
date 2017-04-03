---
layout: post
title: Artsy's Technology Stack, 2017
date: 2017-03-05
categories: [Technology, eigen, force, gravity]
author: orta
series: Artsy Tech Stack
---

# History

Artsy was [launched in 2012 as the "Art Genome Project"](http://www.nytimes.com/2012/10/09/arts/design/artsy-is-mapping-the-world-of-art-on-the-web.html) and grew exponentially ever since.

By 2014 we had 230,000 works of art from 600 museums and institutions and launched our first business, a subscription service for commercial galleries, bringing over 80,000 works for sale and partnerships with 37 art fairs and a handful of benefit auctions. That year collectors from 82 countries inquired on over $5.5B of art.

By 2015 we doubled our "for sale" inventory and aggregated 4,000 of the world's leading galleries and 60 art fairs. We also launched two new businesses: commercial auctions and online media.

Finally, in 2016 we, again, doubled our paid gallery network size to become the largest gallery network in the world and grew to become the most-read online art publication as our highly engaging editorial traffic ballooned 320%. We also launched a platform to bid in live auctions and a consignments service with most major auction houses.

# The Artsy Business in 2017

In 2017 Artsy has three businesses at various stages of development and revenue: _Listings_, _Auctions_ and _Content_.

* _Listings_: Galleries, Fairs and Institutions subscribe to Artsy for a fee because we bring a very large audience of art collectors and enthusiasts to their virtual doors.
* _Auctions_: Auction houses and charities use Artsy as a sales channel for a commission because collectors want to discover and buy art in a single, central platform that excels at surfacing the art they want from a global market.
* _Content_: Brands pay Artsy to reach the first art audience at scale by enabling evergreen content online and for offline engagement during art world events.

The Artsy team is now 160 employees across three offices in New York, Berlin and London. The Engineering organization is now 29 engineers, including 4 leads, 3 directors and a CTO. In this post, we'd like to comprehensively cover what, and how we make the technical and human sides of Artsy businesses work.

<!-- more -->

# Organizational Structures

In 2016, we [updated the Engineering organization](/blog/2016/03/28/artsy-engineering-organization-stack) to be oriented around product verticals. Since then, web and mobile "practices" have largely been subsumed into these separate product teams. Mobile's increasing reliance on React Native has aligned nicely with web tooling. It no longer made sense to keep the teams separate, so where product teams used to have 2 separate sub-teams of engineers, they've now merged into 1.

The Platform "practice" has remained as a way to coordinate and share work among product teams, as well as monitor and upgrade Artsy's platform over time. Most platform engineers operate from within product teams, while a few focusing on data and infrastructure form a core, dedicated Platform team.

<TODO: insert organizational graph here>

# Artsy Technology Infrastructure

## User Facing

A lot of the user-facing focus includes being able to present interfaces with a quality worthy of art.

What you see today when you go to [www.artsy.net](https://artsy.net) is a website built with [Ezel.js](http://ezeljs.com), which is a boilerplate for [Backbone](http://backbonejs.org) projects running on [Node](https://nodejs.org) and using [Express](http://expressjs.com) and [Browserify](http://browserify.org). We used to have separate projects for mobile and desktop web, but they [are now merged](https://github.com/artsy/force/pull/890). The combined app is hosted on [Heroku](http://heroku.com) and uses [Redis](http://redis.io) for caching. Assets, including artwork images, are served from [Amazon S3](http://aws.amazon.com/s3) via the [CloudFront CDN](http://aws.amazon.com/cloudfront). This [code is open-source](https://github.com/artsy/force).

What you see today when you open the [Artsy iOS app](https://itunes.apple.com/us/app/artsy-collect-and-bid-on-fine-art-design/id703796080?mt=8) is a mix of Objective-C, Swift and React Native. Objective-C and Swift continue to provide a lot of over-arching cross-View Controller code. While individual representations of Artsy resources tend to be built in React Native. All of our React Native code uses Relay to handle API integration. This [code is open-source](https://github.com/artsy/eigen).

You can also find Artsy on [Alexa](http://alexa.atsy.net) and [Google Home](http://assistant.artsy.net), which are both open-source Node.js applications.

Our core API serves the public facets of our product, many of our own internal applications, and even [some of your own projects](https://developers.artsy.net). It's built with [Ruby](https://www.ruby-lang.org/en), [Rack](http://rack.github.io), [Rails][rails], and [Grape](https://github.com/intridea/grape) serving primarily JSON. The API is hosted on [AWS OpsWorks](http://aws.amazon.com/opsworks) and retrieves data from several [MongoDB](http://www.mongodb.com) databases hosted with [Compose](https://www.compose.io). It also uses [Memcached](http://memcached.org) for caching and [Redis](https://redis.io) for background queues. It runs background jobs with [delayed_job](https://github.com/collectiveidea/delayed_job). We used to employ [Apache Solr](http://lucene.apache.org/solr) and even [Google Custom Search](https://www.google.com/cse) for the many search functions, but have since consolidated on [Elasticsearch](https://www.elastic.co).

Most modern code for both the website and the iOS app use an orchestration layer which is powered by [GraphQL](http://graphql.org). Our GraphQL server is an [Express](http://expressjs.com) app, using [express-graphql](https://github.com/graphql/express-graphql) to provide a single API end-point. The API does not access our data directly, but forwards requests to the core API or other services. We have been migrating shared display logic into the GraphQL server, to make it easier to build consistent clients. This [code is open-source](https://github.com/artsy/metaphysics).

Consistently, our front-end code has moved towards using React across all platforms along with introducing stricter JavaScript languages like TypeScript over CoffeeScript in order to provide better tooling.

We continue to have a [public HAL+JSON API](https://developers.artsy.net) for external developers. This API is in active use for a few production services inside Artsy and the [website is open-source](https://github.com/artsy/doppler), too.

## Partner-Facing

The vast customer-facing business is powered by a Content Management System (CMS) for gallery and institutional partners. This CMS lets them upload and manager gallery shows, fair booths, create artists, and edit artwork metadata. All CMS components talk to our core API. We also have a number of CMS-like internal applications to manage partners, auctions, art genomes, configuring fairs or performing recurrent billing (we use Stripe for storing and charging credit cards and ACH) with invoicing.

CMS applications are based on stable, mature technologies like [Rails](http://rubyonrails.org), [Bootstrap](http://getbootstrap.com), [Turbolinks](https://github.com/turbolinks/turbolinks) and [CoffeeScript](http://coffeescript.org), and gradually adopts modern client-side technologies like [React](https://facebook.github.io/react) and [Browserify](http://browserify.org). They share a lot of common infrastructure.

We have a generic image-processing service in-house, which uses [Rails](http://rubyonrails.org), [Sidekiq](https://github.com/mperham/sidekiq/), [Redis](https://redis.io), and [RMagick](https://github.com/rmagick/rmagick) with [ImageMagick](http://www.imagemagick.org/script/index.php). It receives image processing requests from many Artsy applications and generates thumbnails, tiles and watermarks images on S3.

## Collector-Facing

Collectors inquire on artworks and engage in a conversation with a partner. For this purpose we have built a generic messaging system that manages communications between different parties. It receives messages via API or e-mail, finds or creates a conversation based on the recipients and forwards them to the proper addresses in that conversation. It's doesn't assume anything about the contents of the messages, which makes it a generic system for any type of conversation. The conversations surface to our partners via CMS.

## Running Auctions

Artsy's Auctions business started with charity auctions. Charity auctions are simpler to map digitally: they have less bids, are more free form in terms of bid increments, have less artworks for sale, they happen slower and the people running them are less risk-averse as they tend to be one-off annual events instead of regular occurrences. We therefore modeled these Auctions inside the core API. As we started implementing commercial auctions with a real-time component (we call these "Live Auctions") the differences in the scale of the domain made it worth moving the logic around an auction into a separate micro-service.

The core API for a commercial auction is a Scala micro-service that uses [Akka](http://akka.io) for distributed computing. It stores information in an append-only storage engine, based on Akka Persistence, with a small library we developed called [atomic-store](https://github.com/artsy/atomic-store). Communication with external clients can either be done via a REST API, or via WebSockets powered by Akka Distributed Pub/Sub. People visiting a Live Auction on the web are interacting with a [universal](https://medium.com/@mjackson/universal-javascript-4761051b7ae9#.ev1yd3juy) [React](https://facebook.github.io/react)+[Redux](http://redux.js.org) JavaScript app, served from an [Express](http://expressjs.com) server. Bidders visiting a Live Auction on iOS are interacting with a Swift application built with [Interstellar](https://github.com/JensRavens/Interstellar), [Starscream](https://github.com/daltoniam/starscream) and [SwiftyJSON](https://github.com/SwiftyJSON/SwiftyJSON).

A more detailed overview of the Auctions stack can be found in [The Tech Behind Live Auction Integration](/blog/2016/08/09/the-tech-behind-live-auction-integration).

## Editorial

We've built our own, lean editorial platform, which uses [MongoDB](https://www.mongodb.com) and [Express](http://expressjs.com) to serve a private JSON API consumed by our applications. This [code is open-source](https://github.com/artsy/positron).

## Data Pipeline

Data generally flows from consumer applications and services into [AWS RedShift](https://aws.amazon.com/redshift). We use a set of [rake](https://github.com/ruby/rake) tasks run on [Jenkins](https://wiki.jenkins-ci.org/display/JENKINS/Build+Flow+Plugin) to move data from our several MongoDB and PostgreSQL databases to Redshift via [S3](https://aws.amazon.com/s3). These rake tasks shell out to [psql](https://www.postgresql.org/docs/9.3/static/sql-copy.html) or [mongo-export](https://docs.mongodb.com/manual/reference/program/mongoexport) to generate CSV files for a list of services and upload them to an S3 bucket, then load those CSV files plus others found in that bucket (placed there by other services) into Redshift. If a [Redshift copy](http://docs.aws.amazon.com/redshift/latest/dg/r_COPY.html) fails due to data changes we sample the CSV and generate a working schema from its contents.

We also store application usage data provided by [Segment Warehouses](https://segment.com/warehouses) as well as data from vendors such as [Salesforce](https://www.salesforce.com) and [Sailthru](http://www.sailthru.com).

For production data processing (such as recommendations), large-scale machine learning or even simpler parallel processing such as generating website sitemaps, we have own own Hadoop cluster configured and managed by [Cloudera Manager](https://www.cloudera.com/products/product-components/cloudera-manager.html) and running on EC2. We leverage [Apache Spark](http://spark.apache.org) and [Hadoop](https://www.cloudera.com/products/open-source/apache-hadoop.html) with some [Ooozie](http://oozie.apache.org) workflow scheduling. The same data pipeline that writes data to S3 also pumps data to HDFS with either Ruby code or [Sqoop](http://sqoop.apache.org) and is read by Spark jobs written in Scala using [Hive](https://hive.apache.org). Spark has improved performance and capacity tenfold over our older in-house systems and we will be moving all lengthy processing implemented in Ruby to this system gradually.

## Analytics

For general data access and dashboards we rely on [Looker](https://looker.com). This system empowers all non-engineers to access all of our data. At the time of writing, there are 50 users running 3,500 queries a day against Redshift via Looker. We've found it expedient to pre-compute common denormalized views, and to create our own session rollups from raw pageviews and events for the additional flexibility it gives us in understanding user behavior.

For more in-depth work, we use [Jupyter Notebooks](https://ipython.org/notebook.html) to connect to our Redshift cluster and by default import [pandas](http://pandas.pydata.org), [sci-kit learn](http://scikit-learn.org/stable), and [pyplot](http://matplotlib.org/api/pyplot_api.html) for data analysis.

## Search

We completed our full migration from [Solr](http://lucene.apache.org/solr) to [Elasticsearch](https://www.elastic.co) in the last 18 months, and now use the latter across artsy.net, from all our artwork filter interfaces through to our real-time artwork similarity feature. Elasticsearch gives us high availability clustering features out of the box and easy horizontal scaling.

## Platform Services

As Artsy's business has grown more complex, so has the data and concepts handled by its core API. We've begun supporting certain product areas with separate, dedicated API services, and even extracting existing API domains into separate services when practical. These services tend to expose simple [REST](https://en.wikipedia.org/wiki/Representational_state_transfer)-ful HTTP APIs, maintain separate data sources, and even do their own [authentication](/blog/2016/10/26/jwt-artsy-journey). This has certain advantages:

* Each system can be deployed and scaled independently.
* Each chooses the best-suited languages and technologies for its purpose.
* Code bases remain more focused and developers' cognitive overhead is minimized.

Balancing these out are some very real disadvantages:

* Development must sometimes touch multiple systems.
* Some data is copied between services. These can become out-of-sync, though we always try to have a single _authoritative_ source in such cases.
* Deploys must be coordinated.

At our size and complexity, a single code base is simply impractical. So, we've tried to be consistent in the coding, deployment, monitoring, and logging practices of these services. The more repeatable and disciplined our process, the less overhead is introduced by additional systems.

We've also explored alternate communication patterns, so systems aren't as dependent on each other's APIs. Recently we've begun publishing a stream of interesting data events from our core systems. Other systems can simply subscribe to the notifications they care about, so the source system doesn't need to be concerned about integrating with one more destination. After experimenting with [Kafka](https://kafka.apache.org) but finding it hard to manage, we switched to [RabbitMQ](https://www.rabbitmq.com) for this purpose. To provide consistency when publishing events we have [our own gem](https://github.com/artsy/artsy-eventservice).

## Operations

All our recent AWS infrastructure is configured in code using [Terraform](https://www.terraform.io). This approach has allowed us to quickly replicate entire deployments along with their dependencies and has increased visibility into the state of our infrastructure across our teams. We started developing an open source [Docker](https://www.docker.com) workflow toolkit named [Hokusai](https://github.com/artsy/hokusai) in order to manage a containerized workflow, CI and deployment to [Kubernetes](https://kubernetes.io). Our Kubernetes clusters are managed using [Kops](https://github.com/kubernetes/kops) and similarly provisioned using Terraform. This new workflow is reducing our dependence on Heroku, giving us more flexibility in our deployments and a more efficient use of server resources.

# People and Culture

## Open Source by Default

By the end of 2016, almost every major front-end application at Artsy was [Open Source by Default](http://code.dblock.org/2015/02/09/becoming-open-source-by-default.html). This means looking for reasons for something to be closed-source as opposed to reasons to be open. Our entire working process is done in the open, from developer PRs to QA. This post was also written collaboratively and in the open as you can see [here](https://github.com/artsy/artsy.github.io/pull/325).

Sometimes it makes sense to keep some details private for competitive reasons. We therefore also create a private GitHub repository for front-end teams that require cross-project issues and team milestones. This is done using [ZenHub](https://www.zenhub.com), and is managed by Engineering leads and Product Managers.

## Developer Workflow

Most development workflow tries to mimic large open-source project development where most work happens on forks and is pull-requested into an Artsy repository shared by everyone. Typically an engineer will start a project, application or service and is automatically its benevolent dictator. They will add continuous integration and will ask other engineers on the team to code review everything from day one. Others will join and entire teams may take over. Continuous deployment and other operational infrastructure will get setup early.

In some of our newer apps we have switched to PR based deployments via CIs. In this case, on Artsy's repository, we would have _master_ and _release_ branches where _master_ is the default branch and all the PRs are made to master. Once a PR is reviewed and merged to _master_ it will automatically get deployed on staging. Production deployment is a pull request from _master_ to a _release_ branch, this way we know what commits are going to be deployed in this release. Once this is merged, CI will automatically deploy the _release_ branch to production.

## Slack

Originally the engineering team used IRC, but in 2015 we switched to Slack and encouraged its use throughout the whole company. We're now averaging about 16,000 Slack messages a day inside Artsy.

Slack usage started out small, but as the Artsy team grew, so did the number of locations where people worked. Encouraging people to move from disparate private conversations in different messaging clients to using slack channels has really made it easier to keep people in the loop. It's made it possible to have the serendipitous collaboration you get by overhearing something important nearby physically.

## Global Engineering

While most Engineers live in New York, our Engineering team has now contributors in Berlin, Seattle, Minneapolis and London. We've not shyed away from hiring regardless of locations.

<TODO: remote workers vs. office workers>

To help people know each-other across the company we developed and open-sourced a [team navigator](https://github.com/artsy/team-navigator). We also facilitate weekly meetings between any three people across the company with a tool called [sup](https://github.com/ilyakava/sup).

## Diversity

Artsy Engineering is 28 people, including 3 leads, 2 directors and a CTO. We have 7 women and 7 non-white guys. While our friends in other Engineering organizations often compliment us on the fact that our team is somewhat more diverse than theirs, we recognize that we're very far from where we'd like to be (eg. only 1/5 people in a leadership role is a woman) and therefore we have a lot more work to do.

<TODO: insert stats>

# Closing Remarks

<TODO: >
