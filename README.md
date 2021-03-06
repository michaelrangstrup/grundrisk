
Grundrisk API integration guide to Preliminary screening endpoint.

# 1. The code

## .net core client with security
Dmp.Examples.GrundriskIntegration is the The .net core client is located in the folder of the same name.

It does a full codeflow login and then calls the grundrisk preliminary screening. Remember to look into "1. Security and access to the endpoint" as you would need the client id and secret.

It is located in the folder DMP.examples.GrundriskIntegration and it is also found here:

https://github.com/danmarksmiljoeportal/grundrisk/tree/master/Dmp.Examples.GrundriskIntegration

## How to generate C# API Client from swagger via NSwagStudio
- Download NSwagStudio and document at here: https://github.com/RicoSuter/NSwag/wiki/NSwagStudio
- After installing, using `GrundriskApiClient.nswag` file in `Dmp.Examples.GrundriskIntegration` folder and generate API Client
- There is issue with `Required = Newtonsoft.Json.Required.DisallowNull` when generate the code (https://github.com/RicoSuter/NSwag/issues/1991 ). So, we need to add custom json settings for API Client
```csharp
public partial class GrundriskClient
{
    partial void UpdateJsonSerializerSettings(JsonSerializerSettings settings)
    {
        settings.ContractResolver = new SafeContractResolver();
    }
}

public class SafeContractResolver : DefaultContractResolver
{
    protected override JsonProperty CreateProperty(MemberInfo member, MemberSerialization memberSerialization)
    {
        var jsonProp = base.CreateProperty(member, memberSerialization);
        jsonProp.Required = Required.Default;
        return jsonProp;
    }
}
```
- Refer the example `GrundriskClient.cs` and `GrundriskClient.Setting.cs` in code

## Spa that show data
The code consist of a SPA using Angular which shows how to fetch data and a .net core client and has no security included and is is intended to show how data can be displayed.



# 2. Security and access to the endpoint

In order to communicate with the endpoint `/screenings/preliminary` the code has to use the OAUTH codeflow against the endpoint. That means you have to contact DMP for registration on the DMP useradm for both test and production.

You will need the role miljoe_grundrisk_foreloebigscreening in order to acces the endpoint. 

Please contact Danmarks Mlijøportal's support at support@miljoeportal.dk to get a client id and client secret for authorization.

## API urls

| Environment | Url |Swagger | Swagger definintion|
| ----------- | ---------------- |----|---|
| TEST        |  https://grundrisk-api.test.miljoeportal.dk/preliminaryScreenings | https://grundrisk-api.test.miljoeportal.dk/swagger/ |https://grundrisk-api.test.miljoeportal.dk/swagger/v1/swagger.json |
| DEMO        | https://grundrisk-api.demo.miljoeportal.dk/preliminaryScreenings | https://grundrisk-api.demo.miljoeportal.dk/swagger/ |https://grundrisk-api.demo.miljoeportal.dk/swagger/v1/swagger.json |
| PROD        | https://grundrisk-api.miljoeportal.dk/preliminaryScreenings | https://grundrisk-api.miljoeportal.dk/swagger/ |https://grundrisk-api.miljoeportal.dk/swagger/v1/swagger.json |

## Web urls

| Environment | Url |
| ----------- | ---------------- |
| TEST        |  https://grundrisk.test.miljoeportal.dk/ |
| DEMO        | https://grundrisk.demo.miljoeportal.dk/ |
| PROD        | https://grundrisk.miljoeportal.dk/ |


The production is not ready before July 1. 2020.

# 3. Danmarks Miljøportal's Identity Provider
Danmarks Miljøportal's identity provider supports OpenID Connect, a simple identity layer on top of the OAuth 2.0 protocol, which allows computing clients to verify the identity of an end-user based on the authentication performed by an authorization server, as well as to obtain basic profile information about the end-user in an interoperable and REST-like manner. In technical terms, OpenID Connect specifies a RESTful HTTP API, using JSON as a data format.

OpenID Connect allows a range of clients, including Web-based, mobile, and JavaScript clients, to request and receive information about authenticated sessions and end-users. The specification suite is extensible, supporting optional features such as encryption of identity data, discovery of OpenID Providers, and session management.

OpenID Connect defines a discovery mechanism, called OpenID Connect Discovery, where an OpenID server publishes its metadata at a well-known URL. The discovery documents are available on the following URL's for the test and production environment respectively.

https://log-in.test.miljoeportal.dk/runtime/oauth2/.well-known/openid-configuration

https://log-in.miljoeportal.dk/runtime/oauth2/.well-known/openid-configuration


The identity provider supports the OAuth 2.0 / OpenID Connect flow ``Authorization code``.

The Authorization Code grant type is used by confidential and public clients to exchange an authorization code for an access token.

After the user returns to the client via the redirect URL, the application will get the authorization code from the URL and use it to request an access token.

# 4. The screening flags in the response from the API endpoint `/preliminaryScreenings`

When you receive a response from the preliminary screenings for each compound the following applies:

1. The value can be "flag":9 , which is 1001 in the binary representation.

2. The enums for calculating which flags that we can derive of the flag value, we use the bitwise left operation. The flag enums are like this:

```
    None = 0 << 0,
    Bly_kobber_eller_PAH = 1 << 0,
    Kompleks_geologi = 1 << 1,
    Manglende_modelstof = 1 << 2,
    Svag_geologi = 1 << 3,
    MTBE_fjernet = 1 << 4,
    Boring_uden_ler = 1 << 5,
    Risiko_pga_vandindvinding = 1 << 6,
```

In our example where the value is decimal 9 and 1001 as binary, can we now derive the following:
 1. Bly_Kobber_eller_PAH is lit on as this is the first 1 in the 1001
 2. Svag_geologi is also lit as we have 1 in the end 1001. 


Next is to put at more wording on the flag enum - here do we use this:


- Bly_kobber_eller_PAH
  - Bly, kobber eller PAH fjernet i trin 1
- Kompleks_geologi
  - Område med kompleks geologi
- Manglende_modelstof
  - Et eller flere forureningsstoffer mangler et modelstof
- Svag_geologi
  - Svag datadækning for bestemmelse af dæklagstykkelser 
- MTBE_fjernet
  - MTBE fjernet
- Boring_uden_ler
  - Lokalitet har boring uden ler i en radius af 100-300 m fra kant
- Risiko_pga_vandindvinding
  - Lokalitet udgør en risiko fordi der ligger en vandindvindingsboring indenfor 100 m

# 5. Standard parameters in the response

When you receive a response from the preliminary screenings for each compound there is also an section called standardParameters.

The values have this description:

\- Infiltration
* Infiltration

\- headGradient
* Hydraulisk gradient

\- aquiferDepth 
* Dybde til grundvandsmagasin

\- horizontalHydraulicConductivity
* Hydraulisk konduktivitet

\- porosity
* Porøsitet

\- firstOrderDegradationRate
* Nedbrydningsrate

\- distNearestWaterWell
* Afstand til nærmeste indvinding

\- distNoClay
* Afstand til nærmeste boring uden ler i dæklag

# 6. The premilinary screening and riskassessment web links in the response

The response will show the links:
- `Preliminary screening link: https://grundrisk.demo.miljoeportal.dk/screening/preliminary/location-details/{response.LocationId}`
- `Preliminary riskassessment link: https://grundrisk.demo.miljoeportal.dk/riskassessment/preliminary/location-details/{response.LocationId}`

# 7. Landfills
The preliminary screening will now scan the input for the following activities or pollutioncause in order to determine if the input constitutes a landfill location.
1. Activitites is located here: https://dkjord-api.demo.miljoeportal.dk/api/locations/landfills/activities  

and has currently the codes
```
	['063', '065', '066', '067', '068', '069']
```
2. Pollutioncauses is located here: https://dkjord-api.demo.miljoeportal.dk/api/locations/landfills/pollutioncauses

and has currently the codes
```
	['90.02.00', '90.02.10', '90.02.20']
```
3. If it does match one of the code it will constitute landfill it will get the following compounds added which then will be screened along with the other input.

The compounds that are added are obtained from: https://dkjord-api.demo.miljoeportal.dk/api/locations/landfills/pollutants 

and has currently the values 

| PollutantCode | PollutantName   | PollutantCompoundGroup      | Concentration | ModelCompoundCode | ModelCompoundName |
|---------------|-----------------|-----------------------------|---------------|-------------------|-------------------|
| 380           | Carbon,org,NVOC | Perkolatparametre           | 73000         | 551               | Kem.iltf. COD     |
| 662           | Benzen          | Benzen                      | 8,5           | 662               | Benzen            |
| 1014          | Ammonium-N      | Perkolatparametre           | 59000         | 551               | Kem.iltf.COD      |
| 1511          | Arsen           | Metaller                    | 12,5          | 1511              | Arsen             |
| 2041          | Jern            | Perkolatparametre           | 55000         | 551               | Kem.iltf.COD      |
| 2618          | Trichlorethylen | Chlorerede opløsningsmiddel | 1,1           | 2618              | Trichlorethylen    |
| 2676          | Phenol          | Phenoler                    | 3,2           | 2676              | Phenol            |
| 3105          | Chlorbenzen     | Chlorbenzen                 | 50            | 3105              | Chlorbenzen       |
| 4512          | Mechlorprop     | Pesticider-Phenoxysyrer     | 220           | 4512              | Mechlorprop       |
| 4515          | Atrazin         | Atrazin                     | 3,4           | 4515              | Atrazin           |

4. CompoundOrigin enum:

```
    Dkjord = 0,
    JAR = 1,
    Landfill = 2
```

* If IsLandfill of preliminary screening in the response is true, the list of preliminary screening contains 10 results that have CompoundOrigin is Landfill

# 8. Removal reasons
The removal reasons are set if the Removed parameter is set to true.

At this case the removalreason will have an value like for instance 13.

To translate this into the removal reasons shown on the grundrisk web use this:

	NotRemoved = 0, // default value
        [Description('MTBE fjernet ifm. olie-miljø-pulje'],)]
        Removed_0_1 = 01,
        [Description(" 'MTBE fjernet, grundet aktiviteter vedr. værksted/olie']]
        Removed_0_2 = 02,
        [Description("'Fjernet grundet modelstof NVOC/COD/Ammonium/PAH/Bly/Kobber'])]
        Removed_1_1 = 11,
        [Description(Fjernet grundet stofgruppe PAH/Bly/Kobber']]
        Removed_1_2 = 12,
        [Description("'Forureningsfladen er 0')]
        Removed_1_3 = 13,  
        [Description("'Grundet dæklagets tykkelse forventes det ikke, at forureningsstoffet når grundvandet'],]
        Removed_2_1 = 21,
        [Description('Fjernet grundet bestemte stoffer']
        Removed_2_2 = 22,
        [Description ('GVK er større end koncentration efter vertikal transport']
        Removed_2_3 = 23,
        [Description('MTBE fjernet grundet dæklagstykkelsen]
        Removed_2_4 = 24,
        // Step 3
        [Description('GVK er større end koncentration efter horisontalt transport)]
        Removed_3_1 = 31
		

# 9. CompoundTranslationType enum:
CompoundTranslationType is a enum with these values: Activity, PollutionCause, Pollutant
```
    Activity = 0,
    PollutionCause = 1,
    Pollutant = 2
```

If the result is V1 and from Activity, CompoundTranslationType is Activity

If the result is V1 and from PollutionCause, CompoundTranslationType is PollutionCause

If the result is V2, CompoundTranslationType is Pollutant


# 10. Test input  for preliminary screening 

It produces produces 2 flags  - value 8 and 9 and standard parameters

* Request
```json
{
  "locationNumber": "000-00003",
  "pollutantComponentCodes": [
    "0703",
    "0490"
  ],
  "activities": [
    {
      "activityCode": "999",
      "pollutionCauseCode": "50.20.10"
    },
    {
      "activityCode": "006",
      "pollutionCauseCode": "50.50.00"
    },
    {
      "activityCode": "999",
      "pollutionCauseCode": "25.12.00"
    }
  ],
  "v1ShapeWkts": [
    "POLYGON ((554931.9389 6145817.3598, 554943.7005 6145814.4546, 554943.7377 6145814.4366, 554957.3742 6145803.8961, 554963.2915 6145805.0073, 554963.3599 6145804.9957, 554982.2349 6145794.1282, 554982.2398 6145794.1252, 554983.5714 6145793.2533, 554995.1822 6145842.5366, 554992.8408 6145844.0876, 554965.4258 6145862.191, 554960.6086 6145861.1682, 554948.7069 6145842.9933, 554957.3674 6145837.738, 554957.4005 6145837.5998, 554948.9235 6145823.9228, 554948.7887 6145823.8888, 554939.6435 6145829.1403, 554931.9389 6145817.3598))",
    "POLYGON ((554944.9397 6145837.2277, 554939.7532 6145829.3079, 554948.8044 6145824.1104, 554957.1773 6145837.6194, 554948.5973 6145842.8258, 554944.9397 6145837.2278, 554944.9397 6145837.2277))"
  ],
  "v2ShapeWkts": [
    "POLYGON ((554944.9397 6145837.2277, 554939.7532 6145829.3079, 554948.8044 6145824.1104, 554957.1773 6145837.6194, 554948.5973 6145842.8258, 554944.9397 6145837.2278, 554944.9397 6145837.2277))"
  ]
}

```

* Response (part of the response)

```json
Response (part of the response)
"polygonArea": 78.5398178100586,     "factor": 0,     "compoundName": "Bly",     "industryName": "Servicestationer",     "activityName": "Benzin og olie, salg af", "flag": 9,
```

```json
"standardParameters": {       "infiltration": 100,       "aquiferDepth": 14.34807491,       "headGradient": 0.007,       "lithoCode": 1,       "distNearestWaterWell": 675.1535179323225,       "distNoClay": 669.5346970240031,       "porosity": 0,       "horizontalHydraulicConductivity": 0,       "firstOrderDegradationRate": 0     },
```
For each compound do you get this result: 
```json
{
  "id": "b2d971a3-7e50-4cd2-8433-ac870031ab66",
  "createdAt": "2020-12-04T03:00:58.5102264+00:00",
  "hasV1Polygon": true,
  "hasV2Polygon": true,
  "v1PolygonResult": {
    "type": 0,
    "hasData": true,
    "factor": 303.41983658441023,
    "polygonStatus": 1,
    "reassessment": null,
    "pollutant": "MTBE"
  },
  "v2PolygonResult": {
    "type": 1,
    "hasData": true,
    "factor": 438.2789431697559,
    "polygonStatus": 1,
    "reassessment": null,
    "pollutant": "MTBE"
  },
  "noModelCompounds": false,
  "noScreeningInputs": false,
  "isLandfill": false,
  "preliminaryScreeningResults": [
    {
      "locationId": "ec072d62-d5ad-4f07-ab24-ac870031ab79",
      "preliminaryScreeningId": "b2d971a3-7e50-4cd2-8433-ac870031ab66",
      "createdAt": "2020-12-04T03:01:02.2635272+00:00",
      "status": 2,
      "industry": null,
      "activity": null,
      "compoundName": "Benzin",
      "compoundCasNr": "",
      "qualityCriterion": 9,
      "worstCaseConcentration": 8000,
      "coverThickness": 14.05306862091965,
      "dataQuality": 2,
      "concentrationDownstream": 0.039270165749594695,
      "removed": false,
      "removalReason": 0,
      "logInfo": [
        2,
        5
      ],
      "concTopTables": 13.619023375377187,
      "conc100mGrundRisk": 0.039270165749594695,
      "factor": 0.004363351749954966,
      "flag": 96,
      "polygonType": 1,
      "polygonArea": 163.87679669065588,
      "standardParameters": {
        "infiltration": 100,
        "aquiferDepth": 14.34807491,
        "headGradient": 0.00402756,
        "lithoCode": 1,
        "distNearestWaterWell": 99.99,
        "distNoClay": 238.13659333991328,
        "porosity": 0.3,
        "horizontalHydraulicConductivity": 0.0001,
        "firstOrderDegradationRate": 0.003
      },
      "compoundOrigin": 1,
      "sourceSize": 163.87679669065588,
      "compoundTranslationType": 2,
      "id": "0b2b54c6-aff0-459d-b527-ac870031b937"
    },
    {
      "locationId": "ec072d62-d5ad-4f07-ab24-ac870031ab79",
      "preliminaryScreeningId": "b2d971a3-7e50-4cd2-8433-ac870031ab66",
      "createdAt": "2020-12-04T03:01:02.2635287+00:00",
      "status": 2,
      "industry": null,
      "activity": null,
      "compoundName": "MTBE",
      "compoundCasNr": "1634044",
      "qualityCriterion": 5,
      "worstCaseConcentration": 50000,
      "coverThickness": 14.05306862091965,
      "dataQuality": 2,
      "concentrationDownstream": 2191.3947158487795,
      "removed": false,
      "removalReason": 0,
      "logInfo": [
        2
      ],
      "concTopTables": 50000,
      "conc100mGrundRisk": 2191.3947158487795,
      "factor": 438.2789431697559,
      "flag": 96,
      "polygonType": 1,
      "polygonArea": 163.87679669065588,
      "standardParameters": {
        "infiltration": 100,
        "aquiferDepth": 14.34807491,
        "headGradient": 0.00402756,
        "lithoCode": 1,
        "distNearestWaterWell": 99.99,
        "distNoClay": 238.13659333991328,
        "porosity": 0.3,
        "horizontalHydraulicConductivity": 0.0001,
        "firstOrderDegradationRate": 0
      },
      "compoundOrigin": 1,
      "sourceSize": 163.87679669065588,
      "compoundTranslationType": 2,
      "id": "bf2f3e6c-47b6-414b-9a90-ac870031b937"
    },
	...
  ],
  "locationId": "ec072d62-d5ad-4f07-ab24-ac870031ab79",
  "locationNumber": "000-00003"
}
 ```
