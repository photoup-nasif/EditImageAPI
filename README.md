PhotoUp EditImage API V1.0.0

* [Introduction to Restful API endpoints](#introduction-to-restful-api-endpoints)
* [Introduction to EditImage API](#introduction-to-editimage-api)
* [Image Extension](#image-extension)
* [Exposure Grouping and Add-ons](#exposure-grouping-and-add-ons)
* [Authentication](#authentication)
* [Request and Response](#request-and-response)
* [EditImage Endpoints: Resources](#editimage-endpoints-resources)
* [Parameter Objects](#parameter-objects)
* [Third Party Requirements](#third-party-requirements)

#### Introduction to Restful API endpoints
Endpoints can be access with http verbs: GET, POST, PUT, and DELETE

- GET /home  (get the list of home)
- POST /home (add new home)
- PUT /home/1/rating (update the rating of home with id = 1)
- DELETE /home/1 (delete home with id = 1)

#### Introduction to EditImage API
This API can be used by third party image provider to upload,rate, and ask revisions thus this API version utilizes or supports only POST and PUT methods.

#### Image Extension
Supported image extensions:
dng, jpg, jpeg, png, tiff, tif, nef, cr2, crw, orf, arw, psd, rw2,
nrw, srf, sr2, raf, pef, raw, mrw, k25, kdc, dcr, x3f, mos, rwl, mef,
erf, 3fr, gpr

#### Exposure Grouping and Add-ons
###### Group Types:
- HDR
  - A bracket of images our team will blend in LR-Enfuse or Photomatix. 1 credit per edited image.
- Masking
  - Two images that need to be blended together in Photoshop. This grouping is commonly used for window masking. 2 credits per edited image.
- Blending
  - Three or more images that need to be blended together by hand in Photoshop. 3 credits per final image.
- Panoramic
  - A series of images that need to be stitched together to create one long panoramic image.

###### Add-ons:
- HDR window masking
  - Select when you want PhotoUp to enhance the window views after HDR blending is complete by masking one of the darker exposures into the window frames. (Applicable if the image is in HDR grouping type)
- Flash Shadow Removal 
  - If you’re shooting a single flash shot that has created shadows of ceiling fans and light fixtures, PhotoUp can clone out those shadows for one additional credit.
- Lawn enhancement
  - Used when the lawn is there but is incomplete and discolored and needs to be made to look full and lush again. PhotoUp will do minor patching for free but select lawn enhancement for major patching and greening.
- Lawn Creation
  - PhotoUp will create a new lawn for you where there isn’t one. Often used for new builds.
- Advance Object Removal
  - An object removal that requires 15 to 25 minutes to complete. Often cleaning up messes left in showers, cleaning off the fridge or generally tidying a room.
- Premium Object Removal
  - An object removal that requires more than 25 minutes to complete. Often removing cards from driveways, rugs from floors and any other removal that requires major reconstruction.
- Day to Dusk:
  - PhotoUp will take an exterior image shot during the day and make it look like it was shot at dusk. Cloudy shots without direct sun shadows are appreciated.

#### Authentication
- Third party clients are provided Public Key and Secret Key
- The API client must provide PU-API-Public-Key in the header (The Key here is the Public key)
- The API client must provide PU-API-Timestamp in the header
- The API client must provide PU-API-Signature in the header
 - The signature is made by hashing components concatenated by a dash (-) with hmac sha256
   - The signature components are: Public Key and timestamp

```
        protected function connectToPhotoUp($method, $url , $data = array()) {
            $timestamp = time();
            $path = parse_url($url, PHP_URL_PATH);
            $method = strtoupper($method);

            $signature_components = array(PUBLIC_KEY, $timestamp);
            $signature = hash_hmac("sha256", implode("\n", $signature_components), SECRET_KEY);

            $headers = array(
                "PU-API-Public-Key: ".PUBLIC_KEY,
                "PU-API-Timestamp: ".$timestamp,
                "PU-API-Signature: ".$signature
                
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
- Upon request 
response should be either in status code
200, 201 or 204
 - with or without response data. If there is a response data the
format would always be in
 -  ``` {"message": "success", .....more data if applicable... }```
- On Failure, status code is either in 4xx or 5xx and in format of
-  ``` {"errors": "String of The Error Message" }```

#### EditImage Endpoints: Resources
| Endpoint | Param | Description |
|-|-|-|
| POST /home | | Creates new home. response example: ```{"message":"success"} ```. The success response here does not mean that the upload is completed. It only means that the data is verified and the upload is still under processing status. See Third Party Requirement. |
||home_id | (int) Required. Home id from third party. |
||address | (String) Required. Address used to name this particular home. |
||instructions | (String) Optional. Instructions/notes for editing the given home. |
||priority | (String) Required. Either "Low", "Med" or "High". This is used by PhotoUp when there are 2 homes uploaded from the same third-party and PhotoUp needs to know which is to be prioritize first. In case of multiple same priority address, PhotoUp will prioritize the homes  based on the deadline/image quantity. |
|| timeline | (String) Required. Either "12Hr", "15Hr", "18Hr", "36Hr" or "48Hr". |
|| images | (hash) Required. The list of images to be edited. See Parameter Objects. |
|| attachment_data | (hash) Optional. List of attachments images that can be used as assets/guides for editing. An example would be an image of fire for fire replacement or an image of grass, blue sky, clouds, water, or an example edited image. See Parameter Objects. |
|-|-|-|
| PUT /home/1234/rating || Updates the rating of given home with id = 1234. |
|| rate | (Int) Required. 1-10. The new/final rating. This can be updated by third party if wanted. |
|| editing_feedback | (String) Optional but Required if rate is 5 or below. The feedback from the third party to result of editing. |
|-|-|-|
| POST /home/1/request_revisions || Request revision on a given home. Third party can request all images to be revised or only some images in a particular home. |
|| all_photos | (boolean) Optional. Set if all images should be revised. |
|| all_photo_comment | (String) Optional but required when all_photos is set. |
|| revision_data | (hash) Optional but required when all_photos is not set. List of images and its comments. See Parameter Objects. |
|-|-|-|
| PUT /home/1/retryUpload || When PhotoUp notifies the third party that the upload of some images failed. Third party can request to retry the uploading process. The response would be ```{"message":"success"} ``` which means that PhotoUp has started the retry procedures and will send another notification if the whole process is successful or not. See Third Party Requirement. |
|-|-|-|
| PUT /home/1/failedDelivery || Third party must notify PhotoUp when PhotoUp delivery for edited images is not successful. PhotoUp will then look for the problems in delivery and will send another delivery request to the third party. See third party requirement. |
|-|-|-|
| PUT /home/1/successDelivery || Third party must notify PhotoUp when PhotoUp delivery for edited images is successful. |

#### Parameter Objects
- images - list/hash of images and all its+- necessary data needed for editing
```
"images": {
        [
                {
                        "image_name": "IMG123.dng", //required
                        "image_url": "http://example.com/theimageurl", //required
                        "image_id": 77466, //required unique identifier from third party
                        "image_note":"make this extra shiny",//optional
                        "image_grouping_type": "Masking", //possible values are 'Hdr','Masking','Blending','Panoramic'//required if multi exposure. Must not be set if image is single exposures
                        "image_other_exposures": [//other exposures is required if grouping_type
                                {
                                        "image_name": "IMG124.dng", //required
                                        "image_url":"http://example.com/theimageurl", //required
                                        "image_id": 77467//required unique identifier from third party
                                },
                                {
                                        "image_name": "IMG125.dng", //required
                                        "image_url":"http://example.com/theimageurl", //required
                                        "image_id": 77468//required unique identifier from third party
                                }
                        ],
                        "image_addons": {
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
                        "image_name": "IMG126.dng", //required
                        "image_url": "http://thirdparty.com/theimageurl", //required
                        "image_note":"make grass green",//optional
                        "image_addons": { "lawn_enhancement": true }
                },
                {
                        "image_name": "IMG127.dng", //required
                        "image_url": "http://thirdparty.com/theimageurl", //required
                }
        ]
}
```
- revision_data - list/hash of image ids and its revision comment
```
"revision_data": {
        [
                {
                        "image_id": 77467, //required
                        "image_comment": "Kindly remove the wires"//required
                },
                {
                        "image_id": 1236,
                        "image_comment": "Too dark. Please brighten up."
                }
        ]
}
```
- attachment_data - list/hash of attachment images used for editing
```
"attachment_data": {
        "attachment_comment": "Please  use the fire.jpg in the fireplace(s) and the other image as reference to the general indoor white balance.",
        "attachment_images": ["https://thirdparty.com/the_attachment_url_1", "https://thirdparty.com/the_attachment_url_2"]
}
```


#### Third Party Requirements
The endpoints below are requirement endpoint for the third party. PhotoUp will send the notifications/data to the third party when necessary.

| Short name | Endpoint | Data from PhotoUp | Description |
|-|-|-|-|
| upload home success | PUT /home/12345/upload_success | ```{"delivery_time":"2018-06-20 04:32:54", "credit_cost":45.75, "deliverable_images": 14}```|  When third party adds home via POST /home PhotoUp will either send a success or failed home upload. |
| upload home fails | PUT /home/12345/upload_failed | ```{"errors" : "Image IMG_123 cannot be downloaded." "image_ids":[1234,5678]}```.| |
| PhotoUp delivery | POST /home/1234/submit | see code below | PhotoUp will send the edited version of images in a given home. May contain data only from  revisions if the home is under a revision. In case of failed initial delivery, PhotoUp will fix the issue and will request the same endpoint with the same data for another attempt. |


```
// hash data for /home/1234/submit
{
        "image_output" : [
                {
                        "image_id": 754869,
                        "image_url": "https://www.photoup.net/the_image_url"
                },
                {
                        "image_id": 754870,
                        "image_url": "https://www.photoup.net/the_image_url"
                },
                {
                        "image_id": 754871,
                        "image_url": "https://www.photoup.net/the_image_url"
                },
                {
                        "image_id": 754872,
                        "image_url": "https://www.photoup.net/the_image_url"
                }
        ]
```

Third party responses can be:
```
{"message": "success"}//200 OK
```
```
{"errors": "Cannot download Image(s)", "image_ids":[12345,1234]}// status 4xx
```
