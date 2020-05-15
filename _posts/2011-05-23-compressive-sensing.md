---
title: Compressive Sensing
date: 2011-05-23
layout: post
comments: true
---
## Comparing DNA sequences using `gzip`

I read something somewhere that described a simple data mining technique that
used compression to determine if two strings are similar to each other. I can't
find that article now but I always wanted to try it. The method works by
comparing the size of the compressed versions of the strings to the size of the
strings if they are compressed together. If the strings are similar, the
compressed size will be smaller than the sum of the compressed sizes for the
individual strings. If not, it will be greater.

For example, if you want to compare two strings to see if they are similar,
first compress them separately and get the length of the compressed strings:

```ruby
>> compress("Austin")
=> 9
>> compress("Boston")
=> 8
```

Now compress them together:

```ruby
>> compress("AustinBoston")
=> 12
```

12 is still more than 9 but less than 17 (the sum of separately compressed
sizes). So the strings must be similar.

I wanted to see if this would work with querying DNA databases for similar
genes, so I wrote a [little script](#Source).

I did some quick testing on a 16S database and by golly it seems to work. Even
with partial sequences. I would like to do some benchmarking to see how
accurate it is.

This isn't really useful for searching a database as it still requires a
one-to-one comparison of every sequence. However, gzip may be faster than
Needleman-Wunsch (I couldn't find the Big O for gzip). So, it could be used to
get a similarity score in cases where you don't need to see an actual
alignment.

## Source

```ruby
LEVEL = 9

database = ARGV[0]
query = ARGV[1]

require 'zlib'

# Compression function.
def deflate(string)
   z = Zlib::Deflate.new(LEVEL)
   dst = z.deflate(string, Zlib::FINISH)
   z.close
   dst
end

# Load entire database into memory.
db = Hash.new
File.new(database).read.split(/^>/).each do |record|
  next if record == "" # split makes a blank one for some reason
  record = record.split("\n")
  header, sequence = record[0], record[1..-1]
  db[header] = sequence.join('')
end

# Read query sequence, and compress.
query_sequence = \
  File.new(query).read.split(/^>/)[1].split("\n")[1..-1].join('') # :D


# Find database sequence is greatest reduction in size

def query(query, db)
  query_size = deflate(query).length
  best, winner = 0, ''

  db.each do |k, seq|
    # Compress DB sequence
    solo = deflate(seq).length

    # Compress both, together
    together = deflate(seq + query).length

    # Score
    if together < (solo + query_size)
      score = (solo + query_size - together)/seq.length.to_f # Normalize by subject length
      if score > best
        best = score
        winner = k
      end
    end

  end
  {:score => best, :hit => winner}
end

result = query(query_sequence, db)
puts result[:hit]
```
