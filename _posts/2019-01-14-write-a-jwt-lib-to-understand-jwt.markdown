---
layot: post
title:  "Write a JWT lib by myself to understand JWT"
date:   2019-01-14 16:00:00 -0500
categories: JWT
tags: [jwt, ruby]
comments: true
---

I wrote this class to explain the basic theory of JWT behind the scene, it may help you to understand the JWT better.

```ruby
module SecureMessage
  class Encode
    # data - the entity to be encoded
    # expired_at - must be a number of seconds since Epoch
    # key - to generate the signature
    # algorithem

    # The list of algorithms here
    # https://ruby-doc.org/stdlib-1.9.3/libdoc/openssl/rdoc/OpenSSL/Digest.html#class-OpenSSL::Digest-label-MD2
    # you should to be noticed that some short length algorithems are not safe, like sha1 / MD5
    def initialize(
      data:,
      key: Rails.application.config.secret_token,
      expired_at: (Time.current + 1.day).to_i,
      algorithm: 'sha256'
    )
      @data = data
      @key = key
      @expired_at = expired_at
      @algorithm = algorithm
    end

    # Generate the token
    # The reason that Base64 is chosen here is
    # firstly, it is able to encode the hash to a hex string
    # secondly, it is decodable
    # Since it is not encryption, the data is kind transparent to others.
    def generate
      Base64.encode64(payload.to_json)
    end

    private

    def payload
      {
        data: @data,
        expired_at: @expired_at,
        signature: signature
      }
    end

    # Using HMAC to digest the signature seems to be proper
    # HMAC - Hash-based Message Authentication, it is good for integrity of a message as well as the authenticity.
    # Keep in mind that the signature is not decryptable.
    def signature
      # Have to use hexdigest over digest
      # otherwise, it will get a wrong signature after payload.to_json
      OpenSSL::HMAC.hexdigest(@algorithm, @key, text_to_be_signed)
    end

    def text_to_be_signed
      "#{@data.to_json}#{@expired_at}"
    end
  end

  class Decode
    def initialize(token:, key: Rails.application.config.secret_token, algorithm: 'sha256')
      @key = key
      @algorithm = algorithm
      @token = token
    end

    def get_data_from_token
      return "Invalid signature" unless verify_signature
      return "The token is expired" if expired_at < Time.current.to_i
      data
    end

    private

    # This is the key process of JWT
    # In order to verify the integrity of the received data, it generates a new signature after receiving the data
    # and then compare the new one with the received one
    def verify_signature
      signature_for_comparison == received_signature
    end

    def received_signature
      payload["signature"]
    end

    # Decode the token and convert it to a hash
    def payload
      @payload ||= JSON.parse(Base64.decode64(@token))
    rescue => ex
      raise "Invalid token #{ex.message}"
    end

    # The methods below are to generate the signature by received payload
    # This process is exactly same as the one in Encode class
    def expired_at
      payload["expired_at"]
    end

    def data
      payload["data"]
    end

    def text_to_be_signed
      "#{data.to_json}#{expired_at}"
    end

    def signature_for_comparison
      @signature_for_comparison ||= @OpenSSL::HMAC.hexdigest(@algorithm, @key, text_to_be_signed)
    end
  end
end

# Test
encode = SecureMessage::Encode.new(data: { first_name: 'abc', last_name: 'def' })
token = encode.generate
decode = SecureMessage::Decode.new(token: token)
decode.get_data_from_token #=> { first_name: 'abc', last_name: 'def' }
```