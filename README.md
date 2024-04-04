# **TestMaverick.SDK.Security HMAC Signature Generation**

HMAC (Hash-based Message Authentication Code) signature is a widely used method to authenticate and secure API requests. In this process, a signature is generated for each API request, which is then verified on the server side to ensure the request's authenticity and integrity.

## **Algorithm**

- Serialize the input data into a structured string format.
```js
  string.Format("{0}\_{1}\_{2}\_{3}\_{4}\_{5}\_{6}\_{7}\_{8}",

  inputObject.security.consumerKey,

  inputObject.security.domain,

  inputObject.security.timestamp,

  secretKey,

  JsonSerializer.Serialize(inputObject.request.user),

  JsonSerializer.Serialize(inputObject.request.mode),

  inputObject.security.remoteIpAddress,

  inputObject.request.meta,

  inputObject.userRequest);
```

- Convert the secret key to a byte array using UTF-8 encoding.
- Create an instance of the HMAC-SHA256 cryptographic service provider using the secret key byte array.
- Compute Hash:
  - Add the concatenated data into a memory stream.
  - Compute the hash of the memory stream using the HMAC-SHA256 cryptography.
  - Obtain the hash bytes.
- Convert Hash Bytes to Hexadecimal String:
  - Convert each byte in the hash bytes to a two-character hexadecimal string representation..

## **Process Overview**

The HMAC signature generation process can be summarized as follows:

1. **Data Preparation**: The input data from a SignRequestModel is serialized into a structured string format using the SerialiseData method. This string includes key components like consumer key, domain, timestamp, secret key, and request details.

2. **Hashing**: The SHA-256 hash of the serialized data is calculated using the HMAC-SHA256 algorithm. This is achieved by calling the ComputeSHA256Hash method, which also includes the timestamp if provided.

3. **Signature Assignment**: The computed HMAC signature is assigned to the signRequestModel.security.signature property in the SignRequestModel using the CreateHMACSignature method.

4. **API Request**: The SignRequestModel with the HMAC signature is used to make an API request. The server can verify the request's authenticity and integrity by recalculating the HMAC signature on its end.

## **Models**

### **SignRequestRequestParameterUserDetails**

The SignRequestRequestParameterUserDetails class represents user details used in signing requests for an API. It allows you to specify essential user information, such as user ID, name, email, and clearance level.

##### **Constructors ;**

```js
  public SignRequestRequestParameterUserDetails(string userId, int clearanceLevel, string email = null, string firstName = null, string lastName = null)
```

##### **Parameters**

- **userId** (string, required): The unique user ID.
- **clearanceLevel** (int): The user's clearance level.
- **email** (string, optional): The user's email address.
- **firstName** (string, optional): The user's first name.
- **lastName** (string, optional): The user's last name.

##### **Example**

```js
  // Create a SignRequestRequestParameterUserDetails object with required parameters

  SignRequestRequestParameterUserDetails userDetails = new SignRequestRequestParameterUserDetails("yourUserId", 1);

  // Create a SignRequestRequestParameterUserDetails object with optional parameters

  SignRequestRequestParameterUserDetails userDetails = new SignRequestRequestParameterUserDetails("yourUserId", 1, email: "<john@example.com>", firstName: "John",lastName: "Doe");
```

### **SignRequestRequestParameter**

The SignRequestRequestParameter class represents the request parameters for signing requests when calling an API. It includes information about the signing mode, user details, and metadata associated with the request.

##### **Constructors ;**

```js
  public SignRequestRequestParameter(string mode, SignRequestRequestParameterUserDetails user, string meta)
```

##### **Parameters**

- **mode** (string, required): The signing mode for the request.
- **user** (SignRequestRequestParameterUserDetails, required): User details associated with the request.
- **meta** (string): Additional metadata for the request.

##### **Example**

```js
  // Create a SignRequestRequestParameter object
  SignRequestRequestParameter requestParameter = new SignRequestRequestParameter("yourMode", new SignRequestRequestParameterUserDetails("yourUserId", 1), "yourMeta");
```

### **SignRequestSecurityParameter**

The SignRequestSecurityParameter class represents the security parameters required for signing requests when calling an API. It includes information such as the consumer key, domain, timestamp, signature, and remote IP address.

##### **Constructors ;**

```js
  public SignRequestSecurityParameter(string consumerKey, string domain, long timestamp, string remoteIpAddress)
```

##### **Parameters**

- **consumerKey** (string, required): The consumer key used for authentication.
- **domain** (string, required): The domain associated with the request.
- **timestamp** (long, required): The timestamp representing the time of the request.
- **remoteIpAddress** (string, required): The remote IP address from which the request originates.

##### **Example**

```js
  // Create a SignRequestSecurityParameter object with required parameters
  SignRequestSecurityParameter request = new SignRequestSecurityParameter("yourConsumerKey", "yourDomain", 1234567890, "yourIpAddress");
```

### **SignRequestModel**

The SignRequestModel class represents a complete sign request, including security parameters, request parameters, and an optional user request. It is used when calling an API to initiate a sign request.

##### **Constructors ;**

```js
  public SignRequestModel(SignRequestSecurityParameter security, SignRequestRequestParameter request, string? userRequest = null)
```

##### **Parameters**

- **security** (SignRequestSecurityParameter, required): The security parameters for the sign request.
- **request** (SignRequestRequestParameter, required): The request parameters for the sign request.
- **userRequest** (string, optional): An optional user request associated with the sign request.

##### **Example**

```js
  // Create a SignRequestSecurityParameter object

  SignRequestSecurityParameter security = new SignRequestSecurityParameter("yourConsumerKey", "yourDomain", 1234567890, "yourIpAddress");

  // Create a SignRequestRequestParameter object

  SignRequestRequestParameter request = new SignRequestRequestParameter("yourMode", new SignRequestRequestParameterUserDetails("yourUserId", 1), "yourMeta");

  // Create a SignRequestModel object with required parameters

  SignRequestModel signRequest = new SignRequestModel(security, request);

  // Create a SignRequestModel object with an optional user request

  SignRequestModel signRequestWithUserRequest = new SignRequestModel(security, request, "additional user request");
```

## **Service Classes**

### **HMACService**

The HMACService class provides methods for creating HMAC (Hash-based Message Authentication Code) signatures for sign requests. This service is used for securing and authenticating requests when calling an API.

#### **Methods**

##### **CreateHMACSignature**

```js
  public async Task&lt;SignRequestModel&gt; CreateHMACSignature(SignRequestModel signRequestModel, string secretKey = null)
```

##### **Parameters**

- **signRequestModel** (SignRequestModel, required): The SignRequestModel containing the request parameters to be signed.
- **secretKey** (string, optional): The secret key used for generating the HMAC signature. If not provided, the service uses the default secret key.

##### **Returns**

- **SignRequestModel**: The SignRequestModel with the HMAC signature added to its security parameters.

##### **Example**

```js
  // Create a SignRequestModel object

  SignRequestModel signRequest = new SignRequestModel(security, request);

  // Create an HMACService object

  HMACService hmacService = new HMACService();

  // Generate an HMAC signature for the sign request

  SignRequestModel signedRequest = await hmacService.CreateHMACSignature(signRequest, "yourSecretKey");
```

##### **SerialiseData**

The SerialiseData method is responsible for serializing input data into a structured string format.

```js
  private string SerialiseData(SignRequestModel inputObject, string secretKey)
```

##### **Parameters**

- **inputObject** (SignRequestModel, required): The SignRequestModel object containing data to be serialized.
- **secretKey** (string, required): The secret key provided by vender.

##### **Returns**

- **serializedInput** (string): combined elements into a single string

##### **Example**

```js
  string serializedInput = string.Format("{0}\_{1}\_{2}\_{3}\_{4}\_{5}\_{6}\_{7}\_{8}",

  inputObject.security.consumerKey,

  inputObject.security.domain,

  inputObject.security.timestamp,

  secretKey,

  JsonSerializer.Serialize(inputObject.request.user),

  JsonSerializer.Serialize(inputObject.request.mode),

  inputObject.security.remoteIpAddress,

  inputObject.request.meta,

  inputObject.userRequest);

  return serializedInput;
```

#####

##### **ComputeSHA256Hash**

The ComputeSHA256Hash method is responsible for calculating the SHA-256 hash of the source data using the HMAC-SHA256 algorithm. It involves several steps to compute the hash.

```js
  private string ComputeSHA256Hash(string sourceData, string secretKey, string timeStamp)
```

##### **Parameters**

- **sourceData** (string, required): The data for which the hash is computed.
- **secretKey** (string, required): The secret key used for HMAC computation.
- **timeStamp** (string,required): The timestamp to include in the data before hashing which is provided in input SignRequestModel

##### **Returns**

- **serializedInput** (string): combined elements into a single string

##### **Sample code**

```js
  private string ComputeSHA256Hash(string sourceData,string secretKey, string timeStamp)
  {

      byte[] secretKeyInByte = Encoding.UTF8.GetBytes(secretKey);

      HMACSHA256 cryptoServiceProvider = new HMACSHA256(secretKeyInByte);

      byte[] hashBytes = ComputeHash(sourceData, cryptoServiceProvider, timeStamp);

      StringBuilder stringBuilder = new StringBuilder();

      foreach (byte b in hashBytes)
      {
        stringBuilder.Append(b.ToString("x2"));
      }
      return stringBuilder.ToString();
  }
```

#####

##### **Method Explanation**

1. Secret Key Conversion:

    ```js
      byte[] secretKeyInByte = Encoding.UTF8.GetBytes(secretKey);
    ```
    - The secret key is converted to a byte array using UTF-8 encoding for compatibility.:

2. HMAC-SHA256 Initialization:

    ```js
      HMACSHA256 cryptoServiceProvider = new HMACSHA256(secretKeyInByte);
    ```

   - An HMAC-SHA256 cryptographic service provider is created with the secret key.

3. Hash Calculation:

    ```js
      byte[] hashBytes = ComputeHash(sourceData, cryptoServiceProvider, timeStamp);
    ```

    - The ComputeHash method is called to compute the hash from the source data.

4. Hexadecimal Conversion:

      ```js
        StringBuilder stringBuilder = new StringBuilder();
        foreach (byte b in hashBytes)
        {
            stringBuilder.Append(b.ToString("x2"));
        }
        return stringBuilder.ToString();
      ```

    - Returns The computed hash bytes are converted to a hexadecimal string.

##### **ComputeHash**

The ComputeHash method is a critical part of the HMAC signature generation process. It is responsible for computing the hash of the source data using the provided cryptographic service provider (HMAC-SHA256). This method is used in the ComputeSHA256Hash method to generate the actual hash.

```js
  private byte[] ComputeHash(string sourceData, HMACSHA256 cryptoServiceProvider, string timeStamp)
```

##### **Parameters**

- **sourceData** (string, required): The data for which the hash is computed.
- **cryptoServiceProvider** (HMACSHA256, required): The HMAC-SHA256 cryptographic service provider for hash computation.
- **timeStamp** (string,required): The timestamp to include in the data before hashing which is provided in input SignRequestModel

##### **Sample code**

```js
  private byte[] ComputeHash(string sourceData, HMACSHA256 cryptoServiceProvider, string timeStamp)
  {
      if (!string.IsNullOrEmpty(timeStamp))
      {
        throw new ArgumentNullException(nameof(timeStamp));
      }

      DataContractSerializer serializer = new DataContractSerializer(sourceData.GetType());

      using (MemoryStream memoryStream = new MemoryStream())
      {
          serializer.WriteObject(memoryStream, sourceData);
          byte[] hashValue = cryptoServiceProvider.ComputeHash(memoryStream.ToArray());
          return hashValue;
      }
  }
```

#####

##### **Method Explanation**

1. Serialization and Hashing:

```js
  DataContractSerializer serializer = new DataContractSerialize (sourceData.GetType());
  using (MemoryStream memoryStream = new MemoryStream())
  {
    serializer.WriteObject(memoryStream, sourceData);
    byte[] hashValue = cryptoServiceProvider.ComputeHash(memoryStream.ToArray());
    return hashValue;
  }
```

- This code block serializes the input data, which may include the timestamp, into a memory stream using a DataContractSerializer. The memory stream is then used to compute the hash using the provided HMAC-SHA256 cryptographic service provider.
