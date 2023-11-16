# Payments with Metatokens
Utilize alternate tokens for the same shopper to ensure a successful payment.

___
On this page, you can find the details of the Metatokens feature, including its benefits and the configurations to implement it.

## Before you begin
This page assumes you're already familiar with:
- [Tokenization](https://docs.adyen.com/online-payments/tokenization).
- [Creation and usage of a token](https://docs.adyen.com/online-payments/tokenization/create-and-use-tokens).
- [Token management](https://docs.adyen.com/online-payments/tokenization/managing-tokens).
- [Payments lifecycle](https://docs.adyen.com/account/payments-lifecycle).

## Overview
Metatokens use a logic similar to tokenization: they replace sensitive data with non-sensitive data. In this case, the difference is that all the shopper’s existing tokens are gathered under an unique string of numbers, called _Metatoken_. The Metatoken will be used during a payment to verify which token provides the best chance of a successful operation.
![image1](https://drive.google.com/uc?id=1I_1D52gbWAcsty9TZK0xygBB5U-bjPwI)

## Benefits of Metatokens
- Leverage all the benefits of [tokenization](https://docs.adyen.com/online-payments/tokenization) and scale them: they enable the use of more than one token for the same shopper.
- Increase the chances of a successful payment right from the beginning: the token with the highest probability of succeeding is used first.
- Allow tokens to be shared across multiple shops under the same company, which reduces bureaucracy and saves time.
- Increase the recovery of failed tokenized payments by 5 to 15% (based on pilot's estimates), which is a significant raise in rescued revenue, especially for those with a subscription-based business model.

## How it works
The process below describes how Metatokens work:
1. Once all shopper's tokens are gathered under a single Metatoken, the payment process starts with the token more likely to result in success. Adyen's machine learning algorithm defines the order in which each token is used, considering the probability of leading to a successful payment.
2. In case of failure, a second attempt is performed, using the second-ranking token (under the same Metatoken). 
3. The process is then repeated, trying each token, until the payment is successful.
4. If no token is valid, the transaction fails.

![image2](https://drive.google.com/uc?id=11AJ5tLNJg_R4Jzmyv5oIULUR_imiFEk5)

There are two basic API-related steps to implement this functionality: 
1. Create a Metatoken.
2. Perform a payment using the Metatoken.

### Step 1: Create a Metatoken
Follow the API configuration below to create a Metatoken.

#### API request
From your server, make a POST **/metaTokens** request and make sure to include the following values:

| Parameter name     | Required | Description |
| ------------------ | -------- | ----------- |
| _shopperReference_ | yes      | Your reference to uniquely identify this shopper. The length of this field should be more than 3 characters. |
| _merchantAccount_  | yes      | The merchant account identifier with which you want to process the transaction. |

Check the sample below: 
```
curl https://checkout-test.adyen.com/v68/metaTokens \
-H "X-API-key: [Your API Key here]" \
-H "Content-Type: application/json" \
-d '{
    "shopperReference": "YOUR_SHOPPER_REFERENCE",
    "merchantAccount": "YOUR_MERCHANT_ACCOUNT"
}'
```

#### API response
Metatoken generation may take between 2 and 30 minutes (depending on the technical conditions). If the Metatoken creation is successful, you will get a response with:

- **merchantAccount**: YOUR_MERCHANT_ACCOUNT.
- **shopperReference**: YOUR_SHOPPER_REFERENCE.
- **metaTokenId**: the 8-digit Metatoken for the future payment details. You will need this to make future payments for the shopper.

Check the sample below:
```
{
    "merchantAccount": "YOUR_MERCHANT_ACCOUNT",
    "shopperReference": "YOUR_SHOPPER_REFERENCE",
    "metaTokenId": "XXXX-XXXX"
}
```
#### Webhook notification
The API response also triggers webhooks notifications. In case of _success_, it will be similar to the sample below:

```
{
    "live":"false",
    "notificationItems":[
        {
            "NotificationRequestItem":{
                "merchantAccount":"YOUR_MERCHANT_ACCOUNT",
                "eventCode":"METATOKEN_CREATION",
                "metaTokenId":"9647-2876",
                "success":"true"
            }
        }
    ]
}
```

**Note**:
- _metaTokenId_: must contain an 8-digit number
- _success_: must be "true".

In case of _failure_, it will be similar to the sample below. This scenario may occur due to a technical error (e.g.: a timeout), and you need to make the API request again.

```
{
    "live":"false",
    "notificationItems":[
        {
            "NotificationRequestItem":{
                "eventCode":"METATOKEN_CREATION",
                "reason":"Technical Error",
                "success":"false"
			}
        }
    ]
}
```

**Note**:
- _metaTokenId_: is not present
- _success_: must be "false"
- _reason_: must be "Technical Error" (only possible value; more informative messages will be added in later versions).

### Step 2: Perform a payment using a Metatoken
Follow the API configuration below to perform a payment using a Metatoken.

#### API request
From your server, make a POST **/payments** request and make sure to include the following value:

| Parameter name | Required | Description |
| -------------- | -------- | ----------- |
| _metaTokenId_  | yes      | The 8-digit Metatoken for the payment details. |

From this point forward, the payment process continues according to [/payments](https://docs.adyen.com/api-explorer/Checkout/latest/post/payments).

## See also
- [API Explorer](https://docs.adyen.com/api-explorer)
- [Webhooks](https://docs.adyen.com/development-resources/webhooks)
- [Glossary](https://docs.adyen.com/get-started-with-adyen/payment-glossary)

## Questions and observations from the Technical Writer
This section includes some observations related to the assignment and questions that I would make in real life to know more about the feature:
- The overall look of the assignment is based on the articles I've seen in the existing Adyen's documentation, including the taxonomy, layout, bulleted text, etc. I attempted to make a similar approach whenever possible.
- Rationale for adopting Metatoken and Metatokens with a capital letter: I used the name of other features (like "Auto Rescue" and "Account Updater") as reference, and it is not a regular noun such as "token". The exceptions are the API commands and fields ("metaToken" and "metaTokenID"), which seem to follow camelCase rules.
- In the provided samples: I corrected some typos and adjusted the indentation of brackets, braces and parentheses.
- In Step 1, it says that the Metatoken creation may take between 2 and 30 minutes to complete. It is a considerable period of time, so it would be a good to explain to the reader when and why these situations occur.
- In Step 2, it would be nice to have samples for the API request/response when performing the payment with a Metatoken. In a real-world situation, since it was not provided, I would ask the developer for them. I could also create a sample myself using the API explorer as source, but the assignment instructions were clear about not needing anything apart from the provided information.
- Clarifications: 
    - How does the system know if the shopper already has one or multiple tokens created? Is the API able to verify that? 
    - After exhausting all the tokens, does it retry all over again? How many times?
    - In case of failure, can this payment be re-retrieved again later?
- Doubt: for example, let's consider that a shopper has 5 tokens under a Metatoken. During a payment, the first 4 fail and only the last one is valid. Although the overall process was successful at the end, will the shopper be notified that the first 4 attempts were unsuccessful and why? What messages will be displayed in those different situations?
