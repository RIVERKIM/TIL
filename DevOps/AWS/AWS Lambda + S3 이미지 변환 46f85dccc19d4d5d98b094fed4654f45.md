# AWS Lambda + S3 이미지 변환

생성일: 2021년 10월 28일 오후 10:44

### Lambda 함수 코드 작성

```jsx
const async = require("async")
const gm = require("gm")
.subClass({imageMagick: true});
const util = require("util");
const { S3Client, GetObjectCommand, PutObjectCommand } = require("@aws-sdk/client-s3")

const s3Client = new S3Client({ region: "us-east-1"});

const transformFunc = process.env.TRANSFORM_FUNC;

exports.handler = function(event, context, callback) {
    var srcBucket = event.Records[0].s3.bucket.name;
    var srcKey = decodeURIComponent(event.Records[0].s3.object.key.replace(/\+/g, " "));  
    var dstBucket = srcBucket + "-output";
    var dstKey    = "output-" + srcKey;

    if (srcBucket == dstBucket) {
        callback("Source and destination buckets are the same.");
        return;
    }

    var typeMatch = srcKey.match(/\.([^.]*)$/);
    if (!typeMatch) {
        callback("Could not determine the image type.");
        return;
    }
    var imageType = typeMatch[1];
    if (imageType != "jpeg" && imageType != "png") {
        callback('Unsupported image type: ${imageType}');
        return;
    }

    // Download the image from S3, transform, and upload to a different S3 bucket.
    console.log("We will first download the image from S3, then transform it, followed by converting it to png and then compressing it before we store it in the destination S3 bucket.");
    async.waterfall([
        function download(next) {
            s3Client.send(new GetObjectCommand({
                Bucket: srcBucket,
                Key: srcKey
            })).then(response => {
                next(null, response);
            });
        },

        function transform(response, next) {
            console.log("before switch");
            switch(transformFunc) {
                case "blur":
                    gm(response.Body).blur(7, 3)
                    .toBuffer(imageType, function (err, buffer) {
                        if(err) {
                            console.log(err);
                            next(err);
                        }else {
                            console.log("blur executed");
                            next(null, response.ContentType, buffer);
                        }
                    })
                    break;
                case "resize":
                    gm(response.Body).resize(300, 200)
                    .toBuffer(imageType, function(err, buffer) {
                        if(err) {
                            next(err);
                        } else {
                            console.log("resize executed");
                            next(null, response.ContentType, buffer);
                        }
                    })
                    break;
                case "thumb":
                    gm(response.Body).thumb(100, 100)
                    .toBuffer(imageType, function (err, buffer) {
                        if(err) {
                            next(err);
                        } else {
                            console.log("thumb executed");
                            next(null, response.ContentType, buffer);
                        }
                    });
                    break;
                default: 
                    console.log("None of the three options were selected");
                    callback("None of the three options were selected");
                    return;
            }
        },
            function upload(contentType, data, next) {
                const uploadParams = {
                    Bucket: dstBucket,
                    Key: dstKey,
                    Body: data,
                    ContentType: contentType
                }
                s3Client.send(new PutObjectCommand(uploadParams)).then(result => {
                    console.log("upload executed");
                    next(null, result);
                })
                
            },
            
            function (err) {
                if (err) {
                    console.error(
                        'Unable to resize ' + srcBucket + '/' + srcKey +
                        ' and upload to ' + dstBucket + '/' + dstKey +
                        ' due to an error: ' + err
                    );
                } else {
                    console.log(
                        'Successfully resized ' + srcBucket + '/' + srcKey +
                        ' and uploaded to ' + dstBucket + '/' + dstKey
                    );
                }
    
                callback(null, "message");
            }
    ])
}
```

### 이벤트 트리거(s3)와 레이어 설정

![Untitled](AWS%20Lambda%20+%20S3%20%E1%84%8B%E1%85%B5%E1%84%86%E1%85%B5%E1%84%8C%E1%85%B5%20%E1%84%87%E1%85%A7%E1%86%AB%E1%84%92%E1%85%AA%E1%86%AB%2046f85dccc19d4d5d98b094fed4654f45/Untitled.png)

- Node_modules는 @aws-sdk/client-s3를 처리하기 위해 아래와 같이 async, gm 을 포함한 모듈들을 포함하고 있다.
    
    ![Untitled](AWS%20Lambda%20+%20S3%20%E1%84%8B%E1%85%B5%E1%84%86%E1%85%B5%E1%84%8C%E1%85%B5%20%E1%84%87%E1%85%A7%E1%86%AB%E1%84%92%E1%85%AA%E1%86%AB%2046f85dccc19d4d5d98b094fed4654f45/Untitled%201.png)
    
- Bucket이 생성될 때 함수가 호출되도록 설정.

### 결과물

왼쪽: 원본, 오른쪽: 사이즈 변경 후

![Untitled](AWS%20Lambda%20+%20S3%20%E1%84%8B%E1%85%B5%E1%84%86%E1%85%B5%E1%84%8C%E1%85%B5%20%E1%84%87%E1%85%A7%E1%86%AB%E1%84%92%E1%85%AA%E1%86%AB%2046f85dccc19d4d5d98b094fed4654f45/Untitled%202.png)

![Untitled](AWS%20Lambda%20+%20S3%20%E1%84%8B%E1%85%B5%E1%84%86%E1%85%B5%E1%84%8C%E1%85%B5%20%E1%84%87%E1%85%A7%E1%86%AB%E1%84%92%E1%85%AA%E1%86%AB%2046f85dccc19d4d5d98b094fed4654f45/Untitled%203.png)