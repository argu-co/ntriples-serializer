# RDF Serializers

<a href="https://travis-ci.org/argu-co/rdf-serializers"><img src="https://travis-ci.org/argu-co/rdf-serializers.svg?branch=master" alt="Build Status"></a>

## About

RDF Serializers enables serialization to RDF formats. It uses your existing [active_model_serializers](https://github.com/rails-api/active_model_serializers) serializers, with a few modifications.
The serialization itself is done by the [rdf](https://github.com/ruby-rdf/rdf) gem.

This was built at [Argu](https://argu.co). If you like what we do, these technologies
or open data in general, send us [a mail](mailto:info@argu.co).

## Installation

Add this line to your application's Gemfile:

```
gem 'rdf-serializers'
```

And then execute:

```
$ bundle
```

## Getting started

First, register the formats you wish to serialize to. For example, add the following to `config/initializers/rdf_serializers.rb`:
```ruby
require 'rdf/serializers/renderers'

RDF::Serializers::Renderers.register(:ntriples)
```
This automatically registers the MIME type.

In your controllers, add:
```ruby
respond_to do |format|
  format.nt { render nt: model }
end
```

## Configuration

Currently, there is one configuration value which can be set using `RDF::Serializers.configure`.
```
RDF::Serializers.configure do |config|
  config.always_include_named_graphs = false # true by default. Whether to include named graphs when the serialization format does not support quads.
end

```

## Formats

You can register multiple formats, if you add the correct gems. For example, add `rdf-turtle` to your gemfile and put this in the initializer:
```ruby
require 'rdf/serializers/renderers'

opts = {
  prefixes: {
    ns:   'http://rdf.freebase.com/ns/',
    key:  'http://rdf.freebase.com/key/',
    owl:  'http://www.w3.org/2002/07/owl#',
    rdfs: 'http://www.w3.org/2000/01/rdf-schema#',
    rdf:  'http://www.w3.org/1999/02/22-rdf-syntax-ns#',
    xsd:  'http://www.w3.org/2001/XMLSchema#'
  }
}

RDF::Serializers::Renderers.register(%i[ntriples turtle], opts)

```

The RDF gem has a list of available [RDF Serialization Formats](https://github.com/ruby-rdf/rdf#rdf-serialization-formats), which includes:
* NTriples
* Turtle
* N3
* RDF/XML
* JSON::LD

and more

## Serializing

Add a predicate to the attributes and relations you wish to serialize.

It's recommended to reuse existing vocabularies provided by the `rdf` gem, and add your own for missing predicates. 
For example:
```
  MY_VOCAB = RDF::Vocabulary.new('http://example.com/')
  SCHEMA = RDF::Vocabulary.new('http://schema.org/')
```

Now add the predicates to your serializers. 

Old: 
```ruby
class PostSerializer < ActiveModel::Serializer
  attributes :title, :body
  belongs_to :author
  has_many :comments
end
```

New:
```ruby
class PostSerializer < ActiveModel::Serializer
  attribute :title, predicate: SCHEMA[:name]
  attribute :body, predicate: SCHEMA[:text]
  belongs_to :author, predicate: MY_VOCAB[:author]
  has_many :comments, predicate: MY_VOCAB[:comments]
end
```

For RDF serialization, you are required to add an `rdf_subject` method to your serializer or model, which must return a `RDF::Resource`. For example:
```ruby
  def rdf_subject
    RDF::URI(Rails.application.routes.url_helpers.comment_url(object))
  end
```

In contrast to the JSON API serializer, this rdf serializers don't automatically serialize the `type` and `id` of your model. 
It's recommended to add `attribute :type, predicate: RDF[:type]` and a method defining the type to your serializers to fix this.

### Custom triples per model

You can add custom triples to the serialization of a model in the serializer, for example:
```ruby
class PostSerializer < ActiveModel::Serializer
  triples :my_custom_triples
  
  def my_custom_triples
    [RDF::Statement.new(RDF::URI('https://example.com'), RDF::TEST[:someValue], 1)]
  end
end
```

### Meta triples

You can add additional triples to the serialization in the controller, for example:
```ruby
render nt: model, meta: [RDF::Statement.new(RDF::URI('https://example.com'), RDF::TEST[:someValue], 1)]
```

## Contributing

The usual stuff. Open an issue to discuss a change, open pull requests directly for bugfixes and refactors.
