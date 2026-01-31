## A/ Fork, Build and Run :
in the browser we search for :

"http://localhost:8080/ws/bank.wsdl"

we see an XML WSDL content 

## B/ Reading the contract :

1/ the XSD file is : src/main/resources/bank.xsd
its role is :

- defines the contract of the SOAP service.
- It specifies the structure of requests and responses (elements, types, constraints).

2/ 

  * GetAccount :

  request :

  GetAccountRequest
- accountId : string

  response :

  GetAccountResponse
- account : AccountType


  * Deposit :

  request :

  DepositRequest
- accountId : string
- amount : decimal

  response :

DepositResponse
- newBalance : decimal


* AccountType :

AccountType
- accountId : string
- owner     : string
- balance   : decimal
- currency  : string


3/ Analyzing the wsdl :

the name space :

<wsdl:definitions targetNamespace="http://example.com/bank">

the PortType :

<wsdl:portType name="BankPort">

it contains these 2 operations :  GetAccount & Deposit 

<wsdl:portType name="BankPort">
<wsdl:operation name="GetAccount">
<wsdl:input message="tns:GetAccountRequest" name="GetAccountRequest"> </wsdl:input>
<wsdl:output message="tns:GetAccountResponse" name="GetAccountResponse"> </wsdl:output>
</wsdl:operation>
<wsdl:operation name="Deposit">
<wsdl:input message="tns:DepositRequest" name="DepositRequest"> </wsdl:input>
<wsdl:output message="tns:DepositResponse" name="DepositResponse"> </wsdl:output>
</wsdl:operation>
</wsdl:portType>



the Endpoint URL :

<soap:address location="http://localhost:8080/ws"/>

the binding :
SOAP over HTTP 


## C/ Postman tests (SOAP)

3/ Sending this POST request "http://localhost:8080/ws" with this raw xml body :

<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
                  xmlns:ban="http://example.com/bank">
  <soapenv:Header/>
  <soapenv:Body>
    <ban:GetAccountRequest>
      <ban:accountId>A100</ban:accountId>
    </ban:GetAccountRequest>
  </soapenv:Body>
</soapenv:Envelope>



and we get this response :

<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/">
    <SOAP-ENV:Header/>
    <SOAP-ENV:Body>
        <ns2:GetAccountResponse xmlns:ns2="http://example.com/bank">
            <ns2:account>
                <ns2:accountId>A100</ns2:accountId>
                <ns2:owner>Alice</ns2:owner>
                <ns2:balance>150.00</ns2:balance>
                <ns2:currency>TND</ns2:currency>
            </ns2:account>
        </ns2:GetAccountResponse>
    </SOAP-ENV:Body>
</SOAP-ENV:Envelope>


Owner: Alice

Balance: 150.00

Currency: TND

4/ sending the same request with this new body :

<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
                  xmlns:ban="http://example.com/bank">
  <soapenv:Header/>
  <soapenv:Body>
    <ban:DepositRequest>
      <ban:accountId>A100</ban:accountId>
      <ban:amount>20.00</ban:amount>
    </ban:DepositRequest>
  </soapenv:Body>
</soapenv:Envelope>


and we get this response :

<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/">
    <SOAP-ENV:Header/>
    <SOAP-ENV:Body>
        <ns2:DepositResponse xmlns:ns2="http://example.com/bank">
            <ns2:newBalance>170.00</ns2:newBalance>
        </ns2:DepositResponse>
    </SOAP-ENV:Body>
</SOAP-ENV:Envelope>


5: if we change the amout to negative :  <ban:amount>-5</ban:amount>
we get this resonse saying that the amout must be > 0 :

<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/">
    <SOAP-ENV:Header/>
    <SOAP-ENV:Body>
        <SOAP-ENV:Fault>
            <faultcode>SOAP-ENV:Client</faultcode>
            <faultstring xml:lang="en">Amount must be &gt; 0</faultstring>
        </SOAP-ENV:Fault>
    </SOAP-ENV:Body>
</SOAP-ENV:Envelope>


## D/ Add one feature:

We’ll choose the WITHDRAW feature :

we update the bank.xsd and add the request and the response (Contract First) :

<xsd:element name="WithdrawRequest">
  <xsd:complexType>
    <xsd:sequence>
      <xsd:element name="accountId" type="xsd:string"/>
      <xsd:element name="amount" type="xsd:decimal"/>
    </xsd:sequence>
  </xsd:complexType>
</xsd:element>

<xsd:element name="WithdrawResponse">
  <xsd:complexType>
    <xsd:sequence>
      <xsd:element name="newBalance" type="xsd:decimal"/>
    </xsd:sequence>
  </xsd:complexType>
</xsd:element>

and also add the Function withdraw(accountId, amount) to the BankService.java :

public BigDecimal withdraw(String accountId, BigDecimal amount) {
  if (amount == null || amount.compareTo(BigDecimal.ZERO) <= 0) {
    throw new InvalidAmountException("Amount must be > 0");
  }

  Account acc = db.get(accountId);
  if (acc == null) {
    throw new UnknownAccountException("Unknown accountId: " + accountId);
  }

  if (acc.balance.compareTo(amount) < 0) {
    throw new InvalidAmountException("Insufficient balance");
  }

  acc.balance = acc.balance.subtract(amount);
  return acc.balance;
}

After that we must edit the file BanlEndpont.java by adding a new endpoint :

  @PayloadRoot(namespace = NAMESPACE_URI, localPart = "WithdrawRequest")
@ResponsePayload
public WithdrawResponse withdraw(@RequestPayload WithdrawRequest request) {
  BigDecimal newBalance =
      bankService.withdraw(request.getAccountId(), request.getAmount());

  WithdrawResponse resp = new WithdrawResponse();
  resp.setNewBalance(newBalance);
  return resp;
}


and after that we test the feature in postman.
****All tests in POSTMAN are grouped and exported as a json collection placed in this repo with the name "SOALab2.postman_collection.json" also screenshoted and put in a directory named postman-screenshots .








