# URL-Shortener 

## Requirements:

### Functional Requirements:

  - Given a URL, our service should generate a shorter and unique alias of it. This is called a short link. This link should be short enough to be easily copied and pasted into applications.
  - When users access a short link, our service should redirect them to the original link.
  - Links will expire after a standard default timespan. Users should be able to specify the expiration time.

### Non Functional Requirements:

  - Shortened links should not be guessable. 

### Extended Requirements:

  - Analytics.
  - The service should also be accessible through a REST API.

## Database Design

  ### Considerations

  1. No relationships between records other than storing which user created a URL, and the association between user and API key.
  2. Using BIGINT identity columns for primary keys. This makes joins, indexes, and lookups much faster than using text keys.
  3. UNIQUE constraints on hash and key_hash columns allow efficient direct searches.
  4. Foreign key columns should have indexes. This dramatically improves queries that fetch all URLs or API keys for a given user.

  ### Database Schema
  
  ```sql
CREATE TABLE URL (
   id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
   hash text NOT NULL UNIQUE,
   originalURL text NOT NULL UNIQUE,
   created_at timestamp DEFAULT now(), 
   expiration_date date,
   user_id bigint NOT NULL REFERENCES User (id)
)

CREATE TABLE User (
   id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
   name text NOT NULL,
   password_hash text NOT NULL,
   email text UNIQUE,
   created_at timestamp DEFAULT now()
)

CREATE TABLE Key (
  id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  user_id bigint NOT NULL REFERENCES User (id) ,
  key_hash text NOT NULL UNIQUE,
  created_at timestamp DEFAULT now(),
  expiration_date date,
  revoked boolean DEFAULT false
);

CREATE INDEX idx_url_user_id ON URL (user_id);
CREATE INDEX idx_key_user_id ON Key (user_id);

```

## Stack:
  - Node.js + Express
  - PostgreSQL
  - Redis

## Design patterns

  1. Strategy pattern: to support multiple ways to generate hashes. 
  2. Cache-Aside Pattern: to implement caching for improved performance.
  3. Oberver Pattern: for analytics and event tracking.

