class: center, middle, inverse

# acceptance test
.footnote[[[ThoughtWorks & Wotif]]()]

---
class: center, middle, inverse

# acceptance test .red[vs] functional test
<!-- functional testing: This is a verification activity; did we build a correctly working product? Does the software meet the business requirements?

For this type of testing we have test cases that cover all the possible scenarios we can think of, even if that scenario is unlikely to exist "in the real world". When doing this type of testing, we aim for maximum code coverage. We use any test environment we can grab at the time, it doesn't have to be "production" caliber, so long as it's usable.

acceptance testing: This is a validation activity; did we build the right thing? Is this what the customer really needs?

This is usually done in cooperation with the customer, or by an internal customer proxy (product owner). For this type of testing we use test cases that cover the typical scenarios under which we expect the software to be used. This test must be conducted in a "production-like" environment, on hardware that is the same as, or close to, what a customer will use. This is when we test our "ilities":

Reliability, Availability: Validated via a stress test.

Scalability: Validated via a load test.

Usability: Validated via an inspection and demonstration to the customer. Is the UI configured to their liking? Did we put the customer branding in all the right places? Do we have all the fields/screens they asked for?

Security (aka, Securability, just to fit in): Validated via demonstration. Sometimes a customer will hire an outside firm to do a security audit and/or intrusion testing.

Maintainability: Validated via demonstration of how we will deliver software updates/patches.

Configurability: Validated via demonstration of how the customer can modify the system to suit their needs.

This is by no means standard, and I don't think there is a "standard" definition, as the conflicting answers here demonstrate. The most important thing for your organization is that you define these terms precisely, and stick to them. -->
---
class: center, middle, inverse

# How much should we write?
![Default-aligned image](images/automated-testing-pyramid.png)

<!-- have a balanced set of automated tests across all levels, with a disciplined approach to having a larger number of smaller specific automated unit/component tests and a smaller number of larger general end-to-end automated tests to ensure all the units and components work together. (My diagram below with attribution). Having just one level of tests, as shown by the stories above, doesn’t work (but if it did I would rather automated unit tests). Just like having a diet of just chocolate doesn’t work, nor does a diet that deprives you of anything sweet or enjoyable (but if I had to choose I would rather a diet of healthy food only than a diet of just chocolate).

Now if we could just convince Salesforce to be more like Airbus and not fly a complete plane (or 50,000 planes) to test everything every-time they make a change and stop David from continuing on his anti-unit pro-system testing anti-intellectual rampage which will result in more damage to our industry than it’s worth.
 -->
---
class: center, middle, inverse

# automated acceptance tests in continuous delivery
![Default-aligned image](images/acceptance-test-stage.png)


---
class: center, middle, inverse
# CD isn't possible without significant levels of automation
![Default-aligned image](images/cd.jpg)

<!-- If your acceptance tests break, that's a sign that you're missing one or more unit tests, so part of CD involves constantly improving your test coverage to try and find bugs earlier in the delivery process where the tests are more fine-grained and the bugs are cheaper to fix. 

- fast feedback loops
- feedback along the deployment pipeline/portability
- reduced testing cycle time
- better time allocation

-->


---
class: center, middle, inverse
# flavors of acceptance tests

---
class: center, middle, inverse
# automated test scripts
.left[
```script
open "http://www.wotif.com"
assertTitle "Wotif.com: Online hotel bookings with instant confirmation"
pause "2000"
clickAndWait "link=Australia"
pause "2000"
clickAndWait "css=#region-1"
pause "3000"
click "//input[@type='submit' and @value='GO']"
assertTitle "Wotif.com: Sydney hotels"
```
]
---
class: center, middle, inverse
# cucumber
.left[
```ruby
Feature: DPS XML Request Verification
  These scenarios cover the full matrix of purchase types that wotif can make. 
  It carries out a purchase or every wotif brand -> card type -> currency and 
  verifies that the produced dps xml contains the expected merchant details

  Background:
    Given dps request xml messages are being trapped
    And No outages exist

  @endpoint
  Scenario: ARN AUD American Express Purchase
    Given I am making a "purchase request"
    When I set the "transRef.orderSource" property to arn
    And I set the "transAmount.currencyCode" property to AUD
    And I set the "creditCard.creditCardType" property to 4
    And I set the "creditCard.creditCardNumber" property to 371111111111114
    Then I post the purchase request
    And the dps request PostUsername is "WotifArnold"
    And the dps request CardHolderName is "Jimmy2 Hendrix2"
    And the dps request CardNumber is "371111111111114"
    And the dps request Amount is "1.00"
    And the dps request DateExpiry is "0117"
    And the dps request Cvc2 is "234"
    And the dps request InputCurrency is "AUD"
```
]

---
class: center, middle, inverse
.left[
```ruby
  @endpoint
  Scenario: ARN AUD Diners Club Purchase
    Given I am making a "purchase request"
    When I set the "transRef.orderSource" property to arn
    And I set the "transAmount.currencyCode" property to AUD
    And I set the "creditCard.creditCardType" property to 5
    And I set the "creditCard.creditCardNumber" property to 36000000000008
    Then I post the purchase request
    And the dps request PostUsername is "WotifArnold"
    And the dps request CardHolderName is "Jimmy2 Hendrix2"
    And the dps request CardNumber is "36000000000008"
    And the dps request Amount is "1.00"
    And the dps request DateExpiry is "0117"
    And the dps request Cvc2 is "234"
    And the dps request InputCurrency is "AUD"

  @endpoint
  Scenario: WTF AUD Visa Purchase
    ...

There are 3215 lines
```
]

---
class: center, middle, inverse
# specification by example

---
class: center, middle, inverse
.left[
```ruby
@endpoint
  Scenario Outline: Purchase for each brand, cardtype and currency combination 
                    where scale is 100
    Given I am making a "purchase request"
    When I set the "transRef.orderSource" property to <ordersource>
    And I set the "transAmount.currencyCode" property to <currencycode>
    And I set the "creditCard.creditCardType" property to <cardtype>
    And I set the "creditCard.creditCardNumber" property to <cardnumber>
    Then I post the purchase request
    And the purchase return code is A150
    And the dps request PostUsername is "<postusername>"
    And the dps request CardHolderName is "Jimmy2 Hendrix2"
    And the dps request CardNumber is "<cardnumber>"
    And the dps request Amount is "1.00"
    And the dps request DateExpiry is "0117"
    And the dps request Cvc2 is "234"
    And the dps request InputCurrency is "<currencycode>"

  Examples:
    | ordersource | currencycode | cardtype |     cardnumber     | postusername |
    |    wtf      |     AUD      |    1     |  4111111111111111  | WotifHotels  |
    |    wtf      |     AUD      |    2     |  5431111111111111  | WotifHotels  |
    |    wtf      |     CAD      |    1     |  4111111111111111  | WotifHotels  |
    |    wtf      |     CAD      |    2     |  5431111111111111  | WotifHotels  |
    |    wtf      |     CHF      |    1     |  4111111111111111  | WotifHotels  |
    |    wtf      |     CHF      |    2     |  5431111111111111  | WotifHotels  |
    |    wtf      |     DKK      |    1     |  4111111111111111  | WotifHotels  |
    |    wtf      |     DKK      |    2     |  5431111111111111  | WotifHotels  |

There are 280 lines 
``` 
]

---
class: center, middle, inverse
# from 
# to


# specifications, not scripts

---
class: center, middle, inverse
# abstract

---
class: center, middle, inverse
# ubiquitous language

---
class: center, middle, inverse
# edge cases

---
class: center, middle, inverse
# key examples

---
class: center, middle, inverse
# end-to-end flows: 1-3 flows

---
class: center, middle, inverse
# accessible

<!-- Specifications, not scripts: she should move to less workflow based scenarios but more specifications about what is needed, as these are easier to understand, more precise and testable;
Abstract: the specification should be abstract enough to highlight the detail, remove the noise, and not being tied to the implementation of the user interface;
Ubiquitous language: the language used by the team and specifications should be consistent throughout the development process to ensure a shared understanding;
Edge cases: unusual variances should be specified to ensure clarity of expectations: “things that seem obvious kill us, if something sounds obvious, that’s where the danger is”;
Key examples: each decision point should have a few key examples, and not more, so it is clear what is expected. These can be created by focussing on the differences between existing scenarios;
End-to-End flows: only a few (1-3) end-to-end flows, not a combination of every decision point combination
Accessible: publishing the specifications so stakeholders can easily access the latest versions. -->

---
class: center, middle, inverse
# living documentation

---
class: center, middle, inverse
# 'three amigos': BA/SME + tester + dev

---
class: center, middle, inverse
# easier to maintain

---
class: center, middle, inverse
# one source of truth

---
class: center, middle, inverse
# executable specifications

---
class: center, middle, inverse
# lemming makes it eaiser
## a service for test data creation

---
class: center, middle, inverse
# integration with lemming

---
class: center, middle, inverse
.left[
```json
{
    "property": {
        "name": "SalesOTADepositTest",
        "displayName": "Wotif SalesOTA Acceptance Test Property-Deposit",
        "countryCode": "AU",
        "accommodationType": "Holiday rental"
    },
    "roomType": {
        "code": "STD",
        "description": "SalesOTA Standard Room",
        "link": "//features//payment//deposit"
    },
    "availability": {
        "startDate": "sod + 1d",
        "endDate": "sod + 5d",
        "rate": 100
    },
    "paymentType":"Deposit"
}
```]

---
class: center, middle, inverse
.left[
```json
{
    "targets": [
        "WOTIF_WEB"
    ],
    "name": "SalesOTATest",
    "displayname": "SalesOTA Acceptance Test Property",
    "countryIsoCode": "AU",
    "inventoryType": "Hotel",
    "ratePlans": [
        {
            "details": {
                "roomType": "STD",
                "description": "SalesOTA Standard Room"
            },
            "rates": [
                {
                    "startDate": "sod + 1d",
                    "endDate": "sod + 5d",
                    "price": 100.0
                }
            ],
            "commissions": [
                {
                    "orderSource": "wtf",
                    "percent" : 0.11
                }
            ]
        }
    ]
}
```]