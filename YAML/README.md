# YAML

* __YAML__ ___"Yet Another Markup Language" , "YAML Ain't Markup Language" : is not Markup language (html,XML)___
 
    * __Markup Language__ ___is a system for annotating a document to define its structure, formatting, and the relationships between its parts (HTML,XML,MARKDOWN)___

* __YAML__ ___is a human-readable data serialization format widely used in configuration files, data exchange, and automation workflows.___

    * __Serialization__ : ___converting data structures or objects into a format (like YAML) that can be stored or transmitted and later reconstructed___

* __Real World Applications:__

    _Kubernetes Resource Definitions_

    _CI/CD Pipelines_

    _Ansible Playbooks_

    _Ansible Playbooks_


## Structure

* __YAML is constructed from three primary data types, or "node types :__ 

    1. __Scalars__

        * __representing single, indivisible data points.__

        * **scalar types** :

            * __strings__ 

                  name: John Doe

            * __integers, floating-point numbers__ 
            
                    age: 30
                    price: 19.99
                    hex_number: 0x1A

            * __booleans__ 

                    is_active: True
                    is_admin: false

            * __null values__

                    last_name: null
                    middle_name: ~

    2. __Mappings "Key-Value Pairs"__

        * __it is an unordered collection of unique key-value pairs and known as a dictionary.__

        * __key is followed by a colon and a space then its value.`key: value`__

                user:
                    name: Jane
                    role: admin


    3. __Sequences "Ordered Lists"__

        * __list or array, it is an ordered series of zero or more items.__

        * __item in a sequence is denoted by a hyphen and a space__

                fruits:
                    - apple
                    - banana
        
## Multi-Line Strings

*  __long strings or blocks of text__

    1. __Literal Block Scalar (|):__
    
        * __preserves all newlines and indentation exactly as they are written.__

                motd: |
                    Welcome to the server!
                    Last updated: 2025-08-17
                    Contact admin@domain.com for issues.

    2. __Folded Block Scalar (>):__
    
        *  __replaces newlines with spaces, effectively "folding" the text into a single line.__

                escription: >
                This is a very long sentence. It spans
                multiple lines in the YAML file for readability,
                but it will be parsed as a single string.

## Anchors (&), Aliases (*), and Merge Keys (<<:)

* __Anchors and Alizses allowing the data to be reused throughout the document without duplication__

1. __Anchors__

    * __Anchor is used to mark a block of data with a name__

2. __Aliases__

    *  __Alias is a reference to that anchored block.__

    * __When the YAML parser reads an alias, it replaces it with the full content of the anchored node it points to__

3. __Merge Keys__

    * __extend existing mappings. It allows a user to "merge" the content of an anchored mapping into a new mapping__
    
    * __The <<: key is followed by an alias (*) that points to a map__

    * __This tells the parser to take all the key-value pairs from the aliased map and insert them into the current map__

            databases:
            default-db: &default-db-settings
                image: postgres:13
                volumes:
                - db-data:/var/lib/postgresql/data
                ports:
                - "5432:5432"

            production:
                <<: *default-db-settings
                container_name: production-db

    * __*the production database will be :*__

            production:
                image: postgres:13
                volumes:
                    - db-data:/var/lib/postgresql/data
                ports:
                    - "5432:5432"
                container_name: production-db


## Explicit Typing (!! Tags)

* __`!!str true` forces the value to be interpreted as a string, and `!!int 0x1C7A` forces a hexadecimal value to be read as an integer.__


## Comments and Document Separation

* __comments using the hash symbol (#)__
* __document is marked by a line with three dashes(---)__

## References

* [yaml.org](https://yaml.org/)

* [yaml.info](https://www.yaml.info/learn/document.html)

* [yaml.irz.fr](https://yaml.irz.fr/)