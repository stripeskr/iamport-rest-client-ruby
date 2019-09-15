# Iamport for Stripes.co.kr

Ruby 사용자를 위한 아임포트 REST API 연동 모듈입니다. (0.3.0 버전에서 stripes 에서 수정하였습니다.)

## Code Status

[![Build Status](https://travis-ci.com/stripeskr/iamport-rest-client-ruby.svg?branch=master)](https://travis-ci.com/stripeskr/iamport-rest-client-ruby)

## 나의 업데이트 후기 0.2.x => 0.3.0
* 코드가 새로운 것이 있나 하고 merge 를 해봄
* 함수명 변경이 좀 있었음 subscribe 라는 단어가 다 빠지고 다음과 같이 존재  
```
def create_payment_again; end
def create_customer; end
```
* spec/iamport_spec.rb:200 에서 pwd_2digit 이 없는데도 테스트 통과
* 즉, 실제 카드정보 넣고 결제를 돌려봐야 함 Project-C 에 테스트 코드 이관 함  
* 부족한거 미비한 것은 채워 넣으면 되는데 함수명을 마음대로 바꾸는 것을 merge 한다는 것은 ... 음

## 결론
```
0.3.0 이상 부터는 iamport 에서 제공하는 것은 안쓰는 걸로
버그잡고 하는 공수가 너무 많이 듬 
```
 

# 세팅하는 방법

```ruby
Iamport.configure do |config|
  config.api_key = "API_KEY"
  config.api_secret = "API_SECRET"
end
```

# 사용법
## token API

```ruby
Iamport.token
```

## payments API

```ruby
Iamport.payment("IMP_UID")
Iamport.payments
Iamport.payments(status: "paid")
Iamport.payments(status: "paid", page: 2)
```

## payments.validation API

##### 결제예정금액을 사전등록
```ruby
Iamport.prepare(merchant_uid: "M00001", amount: 10000)
```

##### 사전등록된 결제정보를 조회
```ruby
Iamport.prepared(merchant_uid: "M00001")
```

## cancel API
body의 값은 [API 문서 - cancel](https://api.iamport.kr/#!/payments/cancelPayment)에 있는 사용하는 것을 추가하여 진행하면 됩니다.​

```ruby
body = {
  imp_uid: "IMP_UID",
  merchant_uid: "M00001",
  amount: ""
}
Iamport.cancel(body)
```

## find API
가맹점지정 고유번호를 이용하여 결제정보를 찾는 API

```ruby
Iamport.find("M00001")
```

## 사용방법

##### 빌링키 발급/변경 요청 예시

```ruby
# create_customer 로 바뀜
customer_uid = "your_customer_1234" 

Iamport.create_customer(
    customer_uid, {
      card_number: "1234-1234-1234-1234",
      expiry: "2019-07",
      birth: "801234",
      pwd_2digit: "00",
      customer_email: "user@your_customer.com",
      customer_name: "홍길동",
      customer_tel: "010-1234-5678"
    }
)
```

&#8251; *필수 항목 : `card_number`, `expiry`, `birth`, `pwd_2digit`*<br />
&#8251; *법인카드의 경우 `pwd_2digit` 항목 생략가능*

##### 빌링키 발급/변경 성공시 Response

```ruby
{"code"=>0,
 "message"=>nil,
 "response"=>
  {"card_name"=>"현대카드",
   "customer_addr"=>nil,
   "customer_email"=>"user@your_customer.com",
   "customer_name"=>"홍길동",
   "customer_postcode"=>nil,
   "customer_tel"=>"010-1234-5678",
   "customer_uid"=>"your_customer_1234",
   "inserted"=>1487921135,
   "updated"=>1487921513}}
```

&#8251; *`inserted`의 값과 `updated`의 값이 같은 경우 신규 발급, 다른 경우 변경을 의미함.*

##### 빌링키 발급/변경 실패시 Response

```ruby
{"code"=>-1,
 "message"=>"카드정보 인증 및 빌키 발급에 실패하였습니다. [F112]유효하지않은 카드번호를 입력하셨습니다. (card_bin 없음)",
 "response"=>nil}
```

##### 발급된 빌링키로 결제
```ruby
res = Iamport.create_payment_again(
      customer_uid: customer_uid,
      merchant_uid: merchant_uid,
      amount: amount,
      name: name,
      buyer_name: 'TEST_NAME',
      buyer_tel: 'TEST_TEL'
)

imp_uid = res['response']['imp_uid']
```

##### 결제 조회 (imp_uid 이용)
```ruby
res = Iamport.payment(imp_uid)

expect(res['code']).to eq(0)
expect(res['response']['imp_uid']).not_to be_nil
expect(res['response']['merchant_uid']).to eq(merchant_uid)
expect(res['response']['amount']).to eq(amount)
expect(res['response']['name']).to eq(name)
expect(res['response']['buyer_name']).to eq('TEST_NAME')
expect(res['response']['buyer_tel']).to eq('TEST_TEL')
```

##### 결제 취소 (imp_uid 이용)
```ruby
res = Iamport.cancel(imp_uid: imp_uid)

expect(res['response']['status']).to eq('cancelled')
```

## 결제 상태만 조회
##### 정상결제 
```ruby
Iamport.payment_status(imp_uid) # ==> 'paid'
```
##### 취소된결제
```ruby
Iamport.payment_status(imp_uid) # ==> 'cancelled'
```
##### 존재하지 않는 imp_uid
```ruby
Iamport.payment_status(imp_uid) # ==> '존재하지 않는 결제정보입니다.'
```


## Installation

Add this line to your application's Gemfile:

```ruby
gem 'httparty'
gem 'iamport'
```

And then execute:

```shell
$ bundle
```

Or install it yourself as:

```shell
$ gem install iamport
```

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake spec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/[USERNAME]/iamport. This project is intended to be a safe, welcoming space for collaboration, and contributors are expected to adhere to the [Contributor Covenant](contributor-covenant.org) code of conduct.


## License

The gem is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).

