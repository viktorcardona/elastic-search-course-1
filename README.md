# elastic-search-course-1


Elastic_Search.txt

https://www.udemy.com/elasticsearch-complete-guide/

Git Hub:
https://github.com/codingexplained/complete-guide-to-elasticsearch

------------------------------------------------------------------------------------------------

**********
ES: Docker:
**********
https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html
https://hub.docker.com/_/elasticsearch
https://www.elastic.co/guide/en/elasticsearch/reference/6.6/docker.html


type:
docker pull docker.elastic.co/elasticsearch/elasticsearch:6.6.2

type:
docker network create esnetwork


type:
docker run -d --name elasticsearch --net esnetwork -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:6.6.2


#Passing multiple environment variables expected by the elasticsearch.yml
type:
docker run -d --name elasticsearch --net esnetwork -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e "cluster.name=theescluster0001" docker.elastic.co/elasticsearch/elasticsearch:6.6.2

.................................
type for starting the container:
docker start elasticsearch
.................................

type:
curl http://localhost:9200

------------------------------------------------------------------------------------------------

******
Kibana:
******
https://www.elastic.co/guide/en/kibana/current/docker.html

type:
docker pull docker.elastic.co/kibana/kibana:6.6.2



type:
docker run -d --name kibana --net esnetwork -p 5601:5601 docker.elastic.co/kibana/kibana:6.6.2

.................................
type for starting the container:
docker start kibana
.................................

type:
http://localhost:5601


Kibana Configuration Options:
https://www.elastic.co/guide/en/kibana/current/settings.html

curl -XPUT "http://elasticsearch:9200/product?pretty"
------------------------------------------------------------------------------------------------

Postman Collection:
Elastic_Search_Testing


------------------------------------------------------------------------------------------------
curl -XPOST "http://elasticsearch:9200/product/default/_bulk" -H 'Content-Type: application/json' -d'
{ "index": { "_id": "100"} }
{ "price": 100 }
{ "index": { "_id": "101"} }
{ "price": 101 }
{ "index": { "_id": "102"} }
{ "price": 103 }
'
------------------------------------------------------------------------------------------------

Import Data Found in test-data.json file into the index product

curl -H "Content-Type: application/json" -XPOST "http://localhost:9200/product/default/_bulk?pretty" --data-binary "@test-data.json"


------------------------------------------------------------------------------------------------
Cluster Status:

curl -XGET "http://elasticsearch:9200/_cat/health?v"

epoch      timestamp cluster          status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1556987076 16:24:36  theescluster0001 yellow          1         1      6   6    0    0        5             0                  -                 54.5%


status=green  It means the cluster is completly functional
status=yellow It means in the above case we are in danger of losing data since there are not replicas of the data
status=red    It means horror. Some data has been lost.

node.total   Total of nodes
shards       Total of shards.  Since we have only one node we cannot have shards, the shards must be in different nodes

------------------------------------------------------------------------------------------------
Mapping:
-------

		It is equivalent of defining a schema for a table in a relational database.

Getting the mapping of the product index:

GET http://localhost:9200/product/default/_mapping

ES does automatic mapping.

The following field description has two mappings:
	1. text:    is used for full-text searches
	2. keyword: is used for exact matches, aggregations and filtering

"description": {
    "type": "text",
    "fields": {
        "keyword": {
            "type": "keyword",
            "ignore_above": 256
        }
    }
}

------------------------------------------------------------------------------------------------

Meta Fields:
-----------

_index
		Contains the name of the index to which a document belongs

_id
		Stores the ID of documents

_source
		
		Contains the original JSON obejct used when indexing a document

_field_names

		Contains the names of every field that contains a non-null value

_routing
		
		Stores the value used to route a documnent to a shard

_version
		
		Stores the internal version of a document

_meta
		
		May be used to store custom data that is left untouched by Elasticsearch
		It is a place where you can store whatever application specific data that you might have

------------------------------------------------------------------------------------------------

Data Types:

	1. Core Data types
		- Text Data Types: Text, used to index full-text value such descriptions.
							Values are analyzed
		- Keyword Data Type: Used for structure data (tags, categories, e-mail addresses,..)
							Not analyzed. 
							Used for:
								filtering and aggregations
		- Numeric Data Types: integer, short, byte, float, double, half_float, scaled_float
		- Date Data Type: represents dates.
		- Boolean Data type: true and false
		- Binary Data Type: accepts a Base64 encoded binary vlaue.
		- Range Data Types: range of values, date ranges or numeric ranges

	2. Complex Data Types
		Data Types that are not primitive, but are more complex, e.g: arrays and objects
			- Object Data Type: Added as JSON bjects; stored as key-value pairs
			- Array Data Type: [1, 2, 3]
			- Array of Objects-
			- Nested Data Type: nested: specialized version of the object data type.
								enables array of objects to be querified independently of each other
								objects values is preserved with the nested data type


	3. Geo Data types
			Used for geographical data
			- Geo-point Data Type: geo_point: Accepts latitude-longitude pairs
			- Geo-shape Data type: geo_shape: used for geographical shapes such as polygons, circles
	
	4. Specialized Data Types
			Data Types with a very specific purpose, e.g. storing IP addresses
			- IP Data Type: ip: Used for storing IPv4 and IPv6 IP addresses
			- Completion Data Type: completion: Used to provide auto-completion "search as you type" functionality.
												Optimized for quick lookups.
												Using suggesters
			- Attachment Data Type: attachment: requires the "Ingest Attachment Processor Plugin"
									Used to make text from various document formats searchable, e.g. PPT, PDF, RTF
									Uses Apache Tika internally for text recognition

------------------------------------------------------------------------------------------------

Getting the mapping of the product index:

GET http://localhost:9200/product/default/_mapping

Add a Mapping:

	Add the discount field with type double for the product index:

PUT http://localhost:9200/product/default/_mapping

{
	"properties": {
		"discount": {
			"type": "double"
		}
	}
}

Existing mappings for fields cannot be updated.
Therefore the following request generates an error:

PUT http://localhost:9200/product/default/_mapping

{
	"properties": {
		"discount": {
			"type": "integer"
		}
	}
}

Solution: delete de index, create new mappings and reindex the dato into the new index.
------------------------------------------------------------------------------------------------

Mappping Parameters

https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-params.html

custom date formats: https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-date-format.html#built-in-date-formats

https://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html

	coerce:
			It converts values to the proper data type, for instance:

				"5"	    -> 5
				"5.0"	-> 5
				5.0		-> 5
	copy_to:
			Enables to build a custom field with fields that we choose

			{
				"first_name" {
					"type": "text",
					"copy_to": "full_name"
				}
			}

	dynamic:
			It can be used to disable dynamic mapping for new fields as you have seen.

	properties:
				Define fields within an "object" or nested field
				It is used to wrap field mappings, both at the top level of documents, but also within inner objects.

	norms:	
			When running search queries, ES does not just determine whether or not a document matches; it also works out how well a document matches. Gives the most releveance results. ES hanlde data that enables calculating relevance scores. ES stores so-called normalization factors for fields that have scoring enabled. These factors are referred to as norms. The norms parameter can be used to disable the storage of this info. Main benefit is to save disk space. Then ES is not able to search given a query by relevance.

			{
				"properties": {
					"full_name": {
						"type": "text",
						"norms": false
					}
				}
			}

	format:
			Defines the format for date fields.

	null_value:

			Replaces NULL values with specified value.

			{
				"properties": {
					"discount": {
						"type": "integer",
						"null_value": 0
					}
				}
			}

	fields:
			Used to index fields in different ways.

------------------------------------------------------------------------------------------------
Defining Custom Date Format 

PUT /product/default/_mapping
{
	"properties": {
		"created": {
			"type": "date",
			"format": "year"
		}
	}
}

PUT /product/default/_mapping
{
	"properties": {
		"created": {
			"type": "date",
			"format": "strict_year"
		}
	}
}


PUT /product/default/_mapping
{
	"properties": {
		"created": {
			"type": "date",
			"format": "strict_date_optional_time||epoch_millis"
		}
	}
}

Custom Format Using Joda Time

PUT /product/default/_mapping
{
	"properties": {
		"created": {
			"type": "date",
			"format": "yyyy/MM/dd HH:mm:ss||yyyy/MM/dd"
		}
	}
}

yyyy/MM/dd HH:mm:ss||yyyy/MM/dd||epoch_milli
------------------------------------------------------------------------------------------------

Note: If disable dynamic mappings to an index.
		Then if we add a new document with a new field discount.
			POST http://localhost:9200/product/default/200
			{
			  "description": "Test",
			  "discount": 20
			}
			Then if we add the mapping of the new Field.
				POST http://localhost:9200/product/default/_mapping
						{
						  "properties": {
							"discount": {
								"type": "integer"
							}
						  }
						}
				The documents can not be found with queries to the new Field.
					POST http://localhost:9200/product/default/_search
						{
						  "query": {
						  	"term" : {
						  		"discount": 20
						  	}
						  }
						}
					The document is not indexed by the new Field.
						This is because when the dynamic mapping is disable and we add new field with its mapping.
							It is required to run a refresh mappings API.
							
								POST /product/_update_by_query?conflicts=proceed

------------------------------------------------------------------------------------------------

Analyzers
---------

		It is an orchestration of the:
			1. Characters Filters: works on the data streams of characters
			2. Tokenizers: Splits the text into terms or tokens
			3. Token Filters: Transform (add, update or remove) tokens

		There is an Standard Analyzer which:
					*****************

			- Divides text into terms using Unicode Text Segmentation (word boundary)
				Removes punctuation
				Lowercases terms
				And optionally removes stop words (e.g: The, A, An, For, In)
			- Example:
					I'm in the mood for drinking semi-dry red wine!
					result:
					i, m, in, the, mood, for, drinking, semi, dry, red, wine

		There is an Simple Analyzer which:
					***************

			- Divides text into terms when encounter a character that is not a letter.
				Also lowercases all terms

		There is an Stop Analyzer which:
					*************

			- Like the simple analizer, but also removes the stop words.
			- Example:
					I'm in the mood for drinking semi-dry red wine!
					result:
					i, m, mood, drinking, semi, dry, red, wine

		There is an Languages Analyzers which:
					*******************

			- Language-specific analyzers, e.g. for English or Spanish
			- The English analizer: 
				- split into words
				- lowercases the terms
				- removes the stop words
				- Convert into the base form of verbs, e.g. Drinking => Drink

		There is a Keyword Analyzer which:
				   ****************

			- No-op analyzer that returns the input as a single term.
				Takes the input and return it as a single term
				It is useful if you do not eant to tokenize or otherwise manipulate the text fields within documents

		There is a Pattern Analyzer which:
				   ****************

			- Uses a regular expression to match token separators and splits text into terms where matches occur
			- Lowercases terms

		There is a Whitespace Analyzer which:
				   *******************

			- Breaks text into terms when encountering a whitespace character.

		Applies only to text fields

		When a document is added an analysis process is executed.
		The process split the text in terms or run a tokenizer then the words ar converted in lowercase, the result is stored in the DB which then is used in order to search data.

	Parts:
	------
		1.	Characters filters: adds or remove characters
				a) One for removing and traslating HTML tags
				b) One for mapping characteres char by char
				c) Last One for mapping characteres applying a regex
		2.	Tokenizer: split text by whitespace and remove symbols such as commas, semicolons, result in array of words (tokens) which preserves the order
				a) Splits herarchical values like file system paths and emits a term for each component
		3.	Token Filters: add, update or remove tokens. Convert tokens into lowercase. Another remove common words such as A, THE, Another identifies sinonyms like good and nice, hose share the same meaning. Therefore, docs with nice are query when searching with the word good.
				a) Standar Token Filter:
						Does not do anything, acts as a placeholder for future versions.
				b) Lowercase Token Filter:
						Normalizes terms to Lowercase
				c) NGram Token Filter:
						Emits n-grams of the specified length based on the provided terms
				d) Stop Token Filter:
						Removes stop words:
							I'm in the mood for drinking
							I'm , mood, drinking
				e) Word Delimiter Token Filter:
						wi-fi: wi, fi
						PowerShell: power, shell
						CE100: CE, 100
						Andy's: Andy
				f) Stemmer Token Filter:
						Reduce words to its base form:
							Drinking: drink
				g) Synomyn Token Filter:
						happy: happy | delighted

	The result of the analyzer is stored in something called:
		Inverted Index


	Inverted Index:
	---------------
					Full text searches are performed over the result of the analyzer and not over the JSON document

					It is a mapping between terms of specific field and the documents with the terms.!

					A cluster will have at least one inverted index


					Term 	|	Doc#1	|	Doc#2
					best 	| 	  x     |      
					recipe 	| 	  x     |      x

Example using the analizer API:
Postman: Elastic_Search_Testing
Postman Requerst: 11_Analyzer_Test1

Request:

curl -X POST \
  http://localhost:9200/_analyze \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -H 'Postman-Token: bb3868ae-fb41-4d31-992e-4069d9fd184d' \
  -d '{
	"tokenizer": "standard",
	"text": "I'\''m in the mood for drinking semi-dry red wine!"
}'

Response:

{
    "tokens": [
        {
            "token": "I'm",
            "start_offset": 0,
            "end_offset": 3,
            "type": "<ALPHANUM>",
            "position": 0
        },
        {
            "token": "in",
            "start_offset": 4,
            "end_offset": 6,
            "type": "<ALPHANUM>",
            "position": 1
        },
        {
            "token": "the",
            "start_offset": 7,
            "end_offset": 10,
            "type": "<ALPHANUM>",
            "position": 2
        },
        {
            "token": "mood",
            "start_offset": 11,
            "end_offset": 15,
            "type": "<ALPHANUM>",
            "position": 3
        },
        {
            "token": "for",
            "start_offset": 16,
            "end_offset": 19,
            "type": "<ALPHANUM>",
            "position": 4
        },
        {
            "token": "drinking",
            "start_offset": 20,
            "end_offset": 28,
            "type": "<ALPHANUM>",
            "position": 5
        },
        {
            "token": "semi",
            "start_offset": 29,
            "end_offset": 33,
            "type": "<ALPHANUM>",
            "position": 6
        },
        {
            "token": "dry",
            "start_offset": 34,
            "end_offset": 37,
            "type": "<ALPHANUM>",
            "position": 7
        },
        {
            "token": "red",
            "start_offset": 38,
            "end_offset": 41,
            "type": "<ALPHANUM>",
            "position": 8
        },
        {
            "token": "wine",
            "start_offset": 42,
            "end_offset": 46,
            "type": "<ALPHANUM>",
            "position": 9
        }
    ]
}


Create the 'existing_analyzer_config' Index with a specific Analyzer and a Filter:

curl -X PUT \
  http://localhost:9200/existing_analyzer_config \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -H 'Postman-Token: 96c80de8-17ad-4366-b71e-6c22d4c3c123' \
  -d '{
  "settings": {
    "analysis": {
      "analyzer": {
        "english_stop": {
          "type": "standard",
          "stopwords": "_english_"
        }
      },
      "filter": {
        "my_stemmer": {
          "type": "stemmer",
          "name": "english"
        }
      }
    }
  }
}'

Test the Analyzer created:

curl -X POST \
  http://localhost:9200/existing_analyzer_config/_analyze \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -H 'Postman-Token: 0bb6dfa6-4dea-46d4-b28e-7f5104127170' \
  -d '{
  "analyzer": "english_stop",
  "text": "I'\''m in the mood for drinking semi-dry red wine!"
}'

Test the Filter created:

curl -X POST \
  http://localhost:9200/existing_analyzer_config/_analyze \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -H 'Postman-Token: 14784439-23b8-45e1-9dd2-5c02be3aefbc' \
  -d '{
  "tokenizer": "standard",
  "filter": [ "my_stemmer" ],
  "text": "I'\''m in the mood for drinking semi-dry red wine!"
}'



Create a custom analyzer named my_analyzer:

curl -X PUT \
  http://localhost:9200/analyzers_test \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -H 'Postman-Token: 70e5908d-551c-4a28-b87b-bf323618dd1c' \
  -d '{
  "settings": {
    "analysis": {
      "filter": {
        "my_stemmer": {
          "type": "stemmer",
          "name": "english"
        }
      },
      "analyzer": {
        "english_stop": {
          "type": "standard",
          "stopwords": "_english_"
        },
        "my_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "char_filter": [
            "html_strip"
          ],
          "filter": [
            "lowercase",
            "trim",
            "my_stemmer"
          ]
        }
      }
    }
  }
}'

Test the analyzer just created:

curl -X POST \
  http://localhost:9200/analyzers_test/_analyze \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -H 'Postman-Token: c490f963-eb26-4caf-a3d2-b312e8f06a45' \
  -d '{
  "analyzer": "my_analyzer",
  "text": "I'\''m in the mood for drinking <strong>semi-dry</strong> red wine!"
}'


Set analyzers to fields (description, teaser) into the analyzers_test index:

curl -X PUT \
  http://localhost:9200/analyzers_test/default/_mapping \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -H 'Postman-Token: dceb2dda-2571-4c9f-81c8-0494a8e1e5b3' \
  -d '{
  "properties": {
    "description": {
      "type": "text",
      "analyzer": "my_analyzer"
    },
    "teaser": {
      "type": "text",
      "analyzer": "standard"
    }
  }
}'

Add a document to the created index:

curl -X POST \
  http://localhost:9200/analyzers_test/default/1 \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -H 'Postman-Token: 0547d98e-1e03-4d3d-95c4-278a96428709' \
  -d '{
  "description": "drinking",
  "teaser": "drinking"
}'


Searching the word match drinking on description does not return results:

curl -X POST \
  http://localhost:9200/analyzers_test/default/_search \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -H 'Postman-Token: c02c8c01-f025-4de1-ad3b-38f1f4b11e1e' \
  -d '{
  "query": {
    "term": {
      "description": {
        "value": "drinking"
      }
    }
  }
}'

Searching the word match drinking on teaser does return results due to the use of the standard analyzer on the teaser field:

curl -X POST \
  http://localhost:9200/analyzers_test/default/_search \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -H 'Postman-Token: c02c8c01-f025-4de1-ad3b-38f1f4b11e1e' \
  -d '{
  "query": {
    "term": {
      "teaser": {
        "value": "drinking"
      }
    }
  }
}'



Adding an Analyzer to an Existing Index
.......................................

	1. Close the index, means the index will blocked for reading and writing operations.
			The index will be shutdown for a period of time.
			It is unfortunate but it is what we have to do


		curl -X POST \
		  http://localhost:9200/analyzers_test/_close \
		  -H 'Cache-Control: no-cache' \
		  -H 'Content-Type: application/json' \
		  -H 'Postman-Token: a3e7877a-f6f8-440c-b4e8-dc74c58674c0'


	2. Add the analyzer using the _settings API:

		curl -X PUT \
		  http://localhost:9200/analyzers_test/_settings \
		  -H 'Cache-Control: no-cache' \
		  -H 'Content-Type: application/json' \
		  -H 'Postman-Token: 78f30aab-75cb-4be0-9ccc-5fe3a13c37f4' \
		  -d '{
			"analysis": {
		      "analyzer": {
		        "french_stop": {
		          "type": "standard",
		          "stopwords": "_french_"
		        }
		      }
		    }
		}'

	2. Open the index

		curl -X POST \
		  http://localhost:9200/analyzers_test/_open \
		  -H 'Cache-Control: no-cache' \
		  -H 'Content-Type: application/json' \
		  -H 'Postman-Token: e5f3bcf8-db96-4c32-b226-59e4dee158d0'



------------------------------------------------------------------------------------------------


------------------------------------------------------------------------------------------------


------------------------------------------------------------------------------------------------


------------------------------------------------------------------------------------------------


------------------------------------------------------------------------------------------------


------------------------------------------------------------------------------------------------


------------------------------------------------------------------------------------------------


------------------------------------------------------------------------------------------------


------------------------------------------------------------------------------------------------


------------------------------------------------------------------------------------------------


------------------------------------------------------------------------------------------------


------------------------------------------------------------------------------------------------


