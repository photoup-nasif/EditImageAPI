PhotoUp EditImage API

[TOC]

* [Introduction to Restful API endpoints](#introduction-to-restful-api-endpoints)

#### Introduction to Restful API endpoints
Endpoints can be access with http verbs: GET, POST, PUT, and DELETE

- GET /home  (get the list of home)
- POST /home (add new home)
- PUT /home/1/rating (update the rating of home with id = 1)
- DELETE /home/1 (delete home with id = 1)

#### Introduction to EditImage API
These API can be used by third party image provider to upload,rate,
and ask revisions thus this API utilizes or supports only POST and PUT
methods.

#### Image Extension
Supported image extensions:
dng, jpg, jpeg, png, tiff, tif, nef, cr2, crw, orf, arw, psd, rw2,
nrw, srf, sr2, raf, pef, raw, mrw, k25, kdc, dcr, x3f, mos, rwl, mef,
erf, 3fr, gpr

#### Authentication
- Third party clients are provided Public Key and Secret Key
- The API client must provide PU-API-PUBLIC-KEY in the header (The Key
here is the Public key)
- The API client must provide PU-API-Timestamp in the header
- The API client must provide PU-API-Signature in the header
 - The signature is made by hashing components concatenated by a dash
(-) with hmac sha256
   - The signature components are: Public Key and timestamp

```
        protected function connectToPhotoUp($method, $url , $data = array()) {
            $timestamp = time();
            $path = parse_url($url, PHP_URL_PATH);
            $method = strtoupper($method);

            $signature_components = array(PUBLIC_KEY, $timestamp);
            $signature = hash_hmac("sha256", implode("\n", $signature_components), SECRET_KEY);

            $headers = array(
            "PU-API-Key: ".INVISO_KEY,
            "PU-API-Signature: ".$signature,
            "PU-API-Timestamp: ".$timestamp
            );
            $ch = curl_init();

            curl_setopt($ch, CURLOPT_URL, $url);
            curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
            curl_setopt($ch, CURLOPT_CUSTOMREQUEST, $method);
            curl_setopt($ch, CURLOPT_RETURNTRANSFER, TRUE);
            curl_setopt($ch, CURLOPT_HEADER, FALSE);
            curl_setopt($ch, CURLOPT_POST, TRUE);

            if(!empty($data)) {
                curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($data));
            }

            $response = curl_exec($ch);
                //do more stuff
        }
```

#### Request and Response
- Media type format
 Both request and response should be in json format
- Upon request successful response should be either in status code
200, 201 or 204
 - with or without response data. If there is a response data the
format would always be in
 -  ``` {"message": "success", .....more data if applicable... }```
- On Failure, status code is either in 4xx or 5xx and in format of
-  ``` {"errors": "String of The Error Message" }```

#### EditImage Endpoints (Resources)
|endpoint| param| note|
|-|-|-|
| POST /home| | Creates new home. response example: ```{"message":"success"} ```. The success response here does not mean that the upload is completed. It only means that the data is verified and the upload is still under processing status. |
||home_id|(int) required. Home id from third party.
||address|(string) required. Address used to name this particular home|
||priority| (string) required. Either Low, Med or High|
||timeline|(String) required. Either 12Hr, 15Hr, 18Hr, 36Hr or 48Hr |
||images|(array) required. The list of images. See Parameter Objects. |
|-|-|-|
|PUT /home/1234/rating|| Updates the rating of given home. 1234 is the id for this example.|
||rate| (Int) required. 1-10. The rating|
|-|-|-|
|/home/1/request/revision|| Request revision on a given home. Third party can request all images to be revised or only some images in a particular home|
||all_photos|(boolean) optional. Set if all images should be revised|
||all_photo_comment|(string) optional but required when all_photos is set|
||revision_data|(array) optional but required when all_photos is not set. List of images and its comments. See Parameter Objects|
|-|-|-|

#### Parameter Objects
- images - list of images and all its+- necessary data needed for editing
```
"images": {
        [
                {
                        "filename": "IMG123.dng", //required
                        "url": "http://example.com/theimageurl", //required
                        "image_id": 77466, //required unique identifier from third party
                        "note":"make this extra shiny",//optional
                        "grouping_type": "Masking", //possible values are 'Hdr','Masking','Blending','Panoramic'//required if multi exposure. Must not be set if image is single exposures
                        "other_exposure": [//other exposures is required if grouping_type
                                {
                                        "filename": "IMG124.dng", //required
                                        "url":"http://example.com/theimageurl", //required
                                        "image_id": 77467//required unique identifier from third party
                                },
                                {
                                        "filename": "IMG125.dng", //required
                                        "url":"http://example.com/theimageurl", //required
                                        "image_id": 77468//required unique identifier from third party
                                }
                        ],
                        "addons": {
                                "hdr_window_mask": true,//optional false by default, and addon_hdr is only available if grouping type is Hdr
                                "flash_shadow_removal": false,//optional false by default
                                "lawn_enhancement": false,//optional false by default
                                "advance_object_removal": true,//optional false by default
                                "lawn_creation": false,//optional false by default
                                "day_to_dusk": false,//optional false by default
                                "premium_object_removal": false,//optional false by default
                        }
                },
                {
                        "filename": "IMG126.dng", //required
                        "url": "http://example.com/theimageurl", //required
                        "note":"make grass green",//optional
                        "addons": { "lawn_enhancement": true }
                },
                {
                        "filename": "IMG127.dng", //required
                        "url": "http://example.com/theimageurl", //required
                }
        ]
}
```
- revision_data - list of image ids and its revision comment
```
"revision_data": {
        [
                {
                        "image_id": 77467, //required
                        "image_comment": "Kindly remove the wires"//required
                },
                {
                        "image_id": 1236
                        "image_comment": "Too dark. Please brighten up."
                }
        ]
}
```

#### Third Party Requirements

|Short name|endpoint|data from PU|note|
|-|-|-|-|
|upload home success |PUT /home/12345/upload_success| | When third party adds home via POST /home PhotoUp will either send a success or failed home upload. |
|upload home fails |PUT /home/12345/upload_failed| ```{"errors" : "File IMG_123 cannot be downloaded."}```| |
|PhotoUp delivery|POST /home/1234/submit|see code below|PhotoUp will send the edited version of images in a given home. May contain data only from  revisions if the home is under a revision.|


```
// data for /home/1234/submit
{
        "image_output" : [
                {
                        "image_id": 754869,
                        "image_id": "https://www.photoup.net/the_image_url"
                },
                {
                        "image_id": 754870,
                        "image_id": "https://www.photoup.net/the_image_url"
                },
                {
                        "image_id": 754871,
                        "image_id": "https://www.photoup.net/the_image_url"
                },
                {
                        "image_id": 754872,
                        "image_id": "https://www.photoup.net/the_image_url"
                }
        ]
```

Third party responses can be:
```
{"message": "success"}//200 OK
```
```
{"errors": "Cannot download file", "files":[12345,1234]}// status 4xx
```
