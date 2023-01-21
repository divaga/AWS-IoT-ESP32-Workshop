# Workshop AWS IoT dengan ESP32

Workshop ini merupakan pengantar untuk Anda yang tertarik menggunakan teknologi Internet of Things. Workshop ini menggunakan AWS Internet of Thing (IoT) sebagai platform utama yang diintegrasikan dengan layanan AWS lainnya seperti AWS Lambda dan Amazon SNS. Device yang digunakan pada workshop  ini menggunakan ESP32 walaupun tipe device yang lain juga dapat digunakan seperti pada user guide AWS IoT disini: https://docs.aws.amazon.com/iot/latest/developerguide/what-is-aws-iot.html


## Kebutuhan yang diperlukan

1. PC/Mac dengan akses Administrator
2. Akun AWS
3. Akses Internet
4. ESP32 board
5. Kabel micro-USB
6. Sensor DHT11 dengan kabel jumper (opsional)

## Step 1: Instalasi dan konfigurasi Arduino IDE


1. Download Arduino IDE disini: https://www.arduino.cc/en/software sesuai dengan platform operating system yang dipakai

2. Navigasi ke menu  **Tools -> Board**, seperti yang Anda lihat semua standard Arduino board akan muncul tetapi untuk ESP32 tidak ada dalam daftar.

3. Buka menu **Preferences** lalu tambahkan link berikut ini di bagian
**Additional Board Manager URLs**.

```
https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
```

<p align="center"> 
<img src="images/ard-ide.jpg" width="80%">
</p>

4. Restart Arduino IDE Anda dan sekarang ESP32 akan muncul di daftar board yang ada.

<p align="center"> 
<img src="images/ard-esp32.jpg" width="80%">
</p>


5. Install library yang diperlukan dengan cara masuk ke menu **"Sketch -> Include Library -> Manage Libraries..."** dan tambahkan library berikut:

    - **MQTT** oleh  Joel Gaehwiler.
    - **ArduinoJson**

<p align="center"> 
<img src="images/ard-lib2.jpg" width="40%">
</p>
<p align="center"> 
<img src="images/ard-lib.jpg" width="40%">
</p>
<p align="center"> 
<img src="images/ntp.jpg" width="40%">
</p>



## Step 2: Instalasi driver untuk komunikasi dengan ESP32


1. Download dan install driver berikut sesuai dengan operating system masing-masing

    MacOS: https://www.silabs.com/documents/public/software/Mac_OSX_VCP_Driver.zip

    Windows: https://www.silabs.com/documents/public/software/CP210x_Universal_Windows_Driver.zip

2. Hubungkan device ESP32 Anda dengan menggunakan kabel micro-USB dan konfigurasi item berikut:
    - Menu **Tools -> Board** and pilih **ESP32 Dev Module**
    - Pastikan nilai setting berikut:
        - Board: **ESP32 Dev Module** 
        - Flash Mode: **QIO** 
        - Flash Frequency: **80MHz** 
        - Flash Size: **4MB (32MB)** 
        - Upload Speed: **921600** 
        - Core Debug Level: **None**

3. Untuk MacOS Eksekusi perintah berikut menggunakan terminal (Untuk sistem operasi Windows silahkan langsung menuju poin nomor 4):

    ```
    ls /dev/tty.*
    ```
    Anda akan melihat device ESP32 Anda terkonfigure dan teridentifikasi sebagai **/dev/tty.SLAB_USBtoUART**

4. Untuk sistem operasi Windows, menggunakan menu **Device Manager**, lihat ke daftar **Ports** dan lihat COM port yang terasosiasi dengan Silicon Labs


<p align="center"> 
<img src="images/win-dev.png" width="40%">
</p>

5. Di Arduino IDE Anda, di menu **Tools** pilih port yang sesuai dengan setup device Anda

6. Naavigasi ke menu **Tools -> Serial Monitor** dan ubah baud rate menjadi **115200**

<p align="center"> 
<img src="images/baud.jpg" width="60%">
</p>


## Step 3: Mencoba pemograman dasar ESP32

1. Di Arduino IDE Anda, buka menu **File -> New** dan copy kode berikut ini:

    ```c
    const int ledPin = 2;

    void setup() {
    pinMode (ledPin, OUTPUT);
    Serial.begin(115200);
    }

    void loop() {
    digitalWrite (ledPin, HIGH);
    Serial.println("ON");
    delay(5000);
    digitalWrite (ledPin, LOW);
    Serial.println("OFF");
    delay(5000);
    }
    ```

2. Upload kode tersebut ke device ESP32 dengan menggunakan menu **Sketch -> Upload**. Pastikan kode dappat terupload sampai muncul informasi **Leaving...**. 

    Jika kode Anda tidak dapat terupload karena error **Wrong boot mode detected** pastikan Anda menekan button **BOOT** pada device ESP32 Anda ketika status pada Arduino IDE Anda menunjukkan status **Connecting...**


3. Anda akan melihat lampu pada board  ESP32 Anda akan menyala dan mati setiap 5 detik. Anda juga dapat melihat informasi logging **OFF/ON** pada Serial Monitor. 

### Selamat!! Anda sudah dapat membuat Sketch pertama Anda, Mengupload kode Anda ke ESP32, berinteraksi dengan LED di ESP32 Anda dan juga menggunakan Serial Monitor.

## Step 4: Konfigurasi AWS IoT

1. Login ke akun AWS Anda, pilih region yang dituju dan pergi ke menu **AWS IoT Core**

<p align="center"> 
<img src="images/iot-main.jpg" width="80%">
</p>

2. Pilih menu **Connect one device** dan isikan nama device Anda di bagian **Thing name**

<p align="center"> 
<img src="images/thing1.jpg" width="80%">
</p>

3. Pilih nilai default pada bagian **Choose platform SDK**

4. Klik pada tombol **Download connection kit** dan download zip file ke local folder Anda

<p align="center"> 
<img src="images/kit.jpg" width="60%">
</p>

5. Lewati langkah **Run connection kit** dan klik Continue

6. Naviagsi  ke menu **Security -> Policies** dan cari policy untuk things yang baru saja Anda buat dan klik pada policy tersebut

7. Pada workshop kali ini, kita akan mengijinkan device yang kita miliki untuk melakukan eksekusi semua perintah pada AWS IoT. Pada  implementasi sebenarnya kita harus membatasi hak akses pada device yang kita miliki sebagai pertimbangan keamanan. Klik pada tombol **Edit version** dan klik pada menu **JSON**

    edit policy document menjadi berikut:

    ```
    {
    "Version": "2012-10-17",
    "Statement": [
        {
        "Effect": "Allow",
        "Action": "iot:*",
        "Resource": "*"
        }
    ]
    }
    ```
    dan klik pada tombol **Save as new version**

8. Pilih versi yang baru dari policy Anda dan klik tombol **Set as active** sehingga versi yang baru menjadi aktif.

<p align="center"> 
<img src="images/active.jpg" width="60%">
</p>

## Step 4: Koneksi  ESP32 ke AWS IoT

1. Unzip file **connect_device_package.zip** yang kita  dapatkan pada step konfigurasi AWS IoT

2. Pada Arduino IDE, klik pada menu tiga titik di kanan lalu pilih **New Tab**

<p align="center"> 
<img src="images/new-tab.jpg" width="60%">
</p>

3. Buat file dengan nama **secrets.h** dengan template berikut:

    ```c
    #include <pgmspace.h>

    #define SECRET
    #define THINGNAME ""

    const char WIFI_SSID[] = "";
    const char WIFI_PASSWORD[] = "";
    const char AWS_IOT_ENDPOINT[] = "xxxxx.amazonaws.com";

    // Amazon Root CA 1
    static const char AWS_CERT_CA[] PROGMEM = R"EOF(
    -----BEGIN CERTIFICATE-----
    -----END CERTIFICATE-----
    )EOF";

    // Device Certificate
    static const char AWS_CERT_CRT[] PROGMEM = R"KEY(
    -----BEGIN CERTIFICATE-----
    -----END CERTIFICATE-----
    )KEY";

    // Device Private Key
    static const char AWS_CERT_PRIVATE[] PROGMEM = R"KEY(
    -----BEGIN RSA PRIVATE KEY-----
    -----END RSA PRIVATE KEY-----
    )KEY";
    ```

4. Isikan informasi berikut pada file **secrets.h**
    - THINGNAME sesuai dengan nama Thing Anda di AWS IoT
    - WIFI_SSID dan WIFI_PASSWORD sesuai dengan koneksi WiFi Anda
    - AWS_IOT_ENDPOINT dapat dilihat di file **start.sh** yang didapat dari download file hasil konfigurasi AWS IoT
    - AWS_CERT_CA dapat dilihat disini: https://www.amazontrust.com/repository/AmazonRootCA1.pem
    - AWS_CERT_CRT dari file ***.cert.pem**
    - AWS_CERT_PRIVATE dari file ***.private.key**


5. Pada file utama Sketch Anda, salin kode berikut. Jangan lupa untuk mengganti **AWS_IOT_PUBLISH_TOPIC** dan **AWS_IOT_SUBSCRIBE_TOPIC** sesuai dengan nama topic Anda.


    ```c
    #include "secrets.h"
    #include <WiFiClientSecure.h>
    #include <MQTTClient.h>
    #include <ArduinoJson.h>
    #include "WiFi.h"
    #include "time.h"

    // The MQTT topics that this device should publish/subscribe
    #define AWS_IOT_PUBLISH_TOPIC   "esp32/pub"
    #define AWS_IOT_SUBSCRIBE_TOPIC "esp32/sub"

    WiFiClientSecure net = WiFiClientSecure();
    MQTTClient client = MQTTClient(256);

    // Bit for sending the MQTT message
    int sendMessageBit = 0;

    // Pin for the button
    const int INTERRUPT_PIN = 0;

    // State for the button press
    volatile byte state = LOW;

    // Get time
    const char* ntpServer = "pool.ntp.org";
    const long  gmtOffset_sec = 25200;
    const int   daylightOffset_sec = 0;


    void connectAWS()
    {
    WiFi.mode(WIFI_STA);
    WiFi.begin(WIFI_SSID, WIFI_PASSWORD);

    Serial.println("Connecting to Wi-Fi");

    while (WiFi.status() != WL_CONNECTED){
        delay(500);
        Serial.print(".");
    }

    // Configure WiFiClientSecure to use the AWS IoT device credentials
    net.setCACert(AWS_CERT_CA);
    net.setCertificate(AWS_CERT_CRT);
    net.setPrivateKey(AWS_CERT_PRIVATE);

    // Connect to the MQTT broker on the AWS endpoint we defined earlier
    client.begin(AWS_IOT_ENDPOINT, 8883, net);

    // Create a message handler
    client.onMessage(messageHandler);

    Serial.print("Connecting to AWS IOT. ");

    while (!client.connect(THINGNAME)) {
        Serial.print(".");
        delay(100);
    }

    if(!client.connected()){
        Serial.println("AWS IoT Timeout!");
        return;
    }

    // Subscribe to a topic
    client.subscribe(AWS_IOT_SUBSCRIBE_TOPIC);

    Serial.println("AWS IoT Connected!");
    }

    void publishMessage(int sendMessageBit)
    {
    StaticJsonDocument<200> doc;
    struct tm timeinfo;
    if(!getLocalTime(&timeinfo)){
        Serial.println("Failed to obtain time");
        return;
    }
    char timeStringBuff[50]; //50 chars should be enough
    strftime(timeStringBuff, sizeof(timeStringBuff), "%A, %B %d %Y %H:%M:%S", &timeinfo);

    doc["time"] =  timeStringBuff;
    doc["message"] = "Hi AWS IoT";
    doc["panic"] = sendMessageBit;
    char jsonBuffer[512];
    serializeJson(doc, jsonBuffer); // print to client

    client.publish(AWS_IOT_PUBLISH_TOPIC, jsonBuffer);
    }

    void messageHandler(String &topic, String &payload) {
    Serial.println("incoming: " + topic + " - " + payload);
    }

    void buttonPress() {
    state = !state;
    if (state == HIGH) {
        Serial.println("Button pressed");
        sendMessageBit = 1;
    }
    if (state == LOW) {
        Serial.println("Button released");
        sendMessageBit = 0;
    }
    }

    void setup() {
    Serial.begin(115200);
    connectAWS();
    //Get Time
    // Init and get the time
    configTime(gmtOffset_sec, daylightOffset_sec, ntpServer);
    pinMode(INTERRUPT_PIN, INPUT_PULLUP);
    // Attach the method to call when the button is pressed
    attachInterrupt(digitalPinToInterrupt(INTERRUPT_PIN), buttonPress, CHANGE);
    }

    void loop() {
    client.loop();
    publishMessage(sendMessageBit);
    delay(1000);
    }
    ```

5. Upload kode Anda ke device dan lihat status pada Arduino IDE Serial Monitor

6. Anda dapat  melihat  topic Anda  dipublish ke AWS Iot dengan melihat di AWS IoT Console pada menu **Test -> MQTT test client**. 
Anda dapat menuliskan nama topic Anda pada bagian Topic filter dan klik tombol Subscribe. Anda dapat melihat topic dikirim dari ESP32 device Anda ke AWS IoT per 1 detik.


<p align="center"> 
<img src="images/topic.jpg" width="80%">
</p>

7. Anda dapat melihat data **panic** akan berubah dari 0 menjadi 1 jika  tombol **BOOT** pada device Anda ditekan

8. Anda juga dapat melakukan publish topic dengan masuk ke  menu **Publish to a topic** dan isikan nama topic SUBSCRIBE Anda dan klik tombol publish. Message yang dikirim akan dapat Anda lihat di Arduino IDE Serial Monitor. 

<p align="center"> 
<img src="images/publish.jpg" width="80%">
</p>

<p align="center"> 
<img src="images/publish-serial.jpg" width="50%">
</p>

## Step 5: Konfigurasi SNS dan Lambda

1. Navigasi ke menu **Simple Notification Service** di AWS Console, buat Standard topic dan catat ARN dari SNS topic Anda.

<p align="center"> 
<img src="images/sns1.jpg" width="80%">
</p>

2. Create Subscription di topic yang Anda buat, pilih "Email" sebagai protokol dan isi alamat email Anda.

<p align="center"> 
<img src="images/sns-sub.jpg" width="80%">
</p>

3. Lakukan konfirmasi dari topic subscription yang anda buat dengan mengklik link yang dikirimkan ke email Anda.

<p align="center"> 
<img src="images/sns-email.jpg" width="50%">
</p>

4. Navigasi ke menu **Lambda** di AWS Console, buat function menggunakan Python 3.9.

<p align="center"> 
<img src="images/lambda.jpg" width="80%">
</p>

5. Grant akses untuk eksekusi SNS dari Lambda dengan masuk ke tab **Configuration** dan klik Role name

<p align="center"> 
<img src="images/lambda-iam.jpg" width="80%">
</p>

6. Window IAM akan terbuka dan klik menu **Add permissions** lalu pilih **Attach policies** dan pilih **AmazonSNSFullAccess** dan klik tombol **Attach policies** untuk menambahkan  role SNS untuk Lambda yang kita buat.

<p align="center"> 
<img src="images/iam-role.jpg" width="80%">
</p>

7. Salin kode berikut kedalam Lambda code dan ganti bagian SNS_TOPIC_ARN sesuai dengan SNS topic yang sudah Anda buat:
    ```python
    import json
    import boto3

    sns = boto3.client("sns")

    def lambda_handler(event, context):
        time_stamp = event["time"]
        panic = event["panic"]
        
        message = "Panic Button pressed on " + time_stamp
        subject = "Panic Button Pressed!"
        topic = "SNS_TOPIC_ARN"
        
        if (panic != 0):
            response = sns.publish(Message=message, Subject=subject,TopicArn=topic )
            
        return {
            'statusCode': 200,
            'body': json.dumps("Lambda Executed")
        }
    ```

## Step 6: Membuat AWS IoT Rule

1. Navigasi ke menu **Manage -> Message Routing -> Rules** di AWS IoT Console dan masukkan nama rule yang akan dibuat

<p align="center"> 
<img src="images/iot-rule.jpg" width="80%">
</p>

2. Salin kode berikut pada bagian **Configure SQL statement** dan ganti bagian **FROM** sesuai dengan nama topic Anda dan klik tombol Next:

    ```sql
    SELECT * FROM 'esp32/pub'
    ```

3. Pada bagian **Rule actions** di halaman **Attach rule actions** pilih Action = Lambda dan pilih Lambda function yang sudah Anda buat lalu klik tombol "Create".

<p align="center"> 
<img src="images/rule-action.jpg" width="80%">
</p>

## Step 7: Test Panic Button

1. Tekan tombol **BOOT** pada ESP32 Anda lalu cek email yang sudah Anda daftarkan di SNS. Anda seharusnya menerima email alert dari SNS.

<p align="center"> 
<img src="images/email.jpg" width="80%">
</p>

### **Selamat !!! Anda sudah dapat melakukan integrasi dari ESP32 ke AWS IoT yang akan mengirimkan email menggunakan AWS Lambda dan Amazon SNS.**




## Step 8: Tantangan!

Tambahkan sensor DHT11 untuk temperatur dan kelembaban dan modifikasi kode ESP32 diatas. Anda juga dapat berkreasi untuk membuat user interface yang menarik dari data tersebut yang masuk ke topic di AWS IoT Core.

## Referensi:

Sebagian besar konten dari workshop ini disadur dari:
https://github.com/aws-samples/aws-iot-workshop dan https://aws.amazon.com/blogs/compute/building-an-aws-iot-core-device-using-aws-serverless-and-an-esp32/


Anda dapat melihat dokumentasi dari AWS IoT disini: https://docs.aws.amazon.com/iot/index.html
