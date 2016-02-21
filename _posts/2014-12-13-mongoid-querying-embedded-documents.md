---
title: 'Mongoid: Querying by Embedded Attributes'
---

At my company we're using [Mongoid](http://mongoid.org/en/mongoid/index.html) to interface with MongoDB in a Ruby on Rails application.  In general this is a pretty high-quality set of Ruby gems (it's built on top of the Moped and Origin gems), but one thing that eluded me and doesn't seem to mentioned in the documentation is how to query for documents by an embedded document attribute.  It turns out to be quite simple:

### 1) Querying by Defined Embedded Fields

Here's a snippet from our code:

```ruby
AccountCalculation.where('campaign_calculations.campaign_id' => campaign.id)
```

`CampaignCalculation` is an "embeds_many" relation on `AccountCalculation`, and each `CampaignCalculaion` has an integer field `campaign_id` defined.

### 2) Querying by Embedded Document IDs

This one was a little tricker, in that the type conversion (from `String` to `BSON::ObjectID` in this case) is not handled automatically.  By default each ID for a MongoID document is of type `BSON::ObjectID`.  Here's a related snippet:

```ruby
OldProject.where('project_segments._id' => BSON::ObjectId.from_string(id_s))
```

I hope these quick tips will save a few people the hassle of trying to figure this out.  Even worse...we originally were finding ourselves getting a large dataset back from our querying and using [Enumerable](http://ruby-doc.org/core-2.1.5/Enumerable.html) methods to query by embedded attributes.  You'll use more memory on your server that way, of course, and it's significantly slower than having MongoDB handle the query.