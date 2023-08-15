# Cloud Key Management Service - Hệ thống Quản lý Khóa

# Overview

- KMS: là 1 hệ thống quản lý khóa mật mã được sử dụng để quản lý vòng đời của các keys, cung cấp một giao diện thống nhất và hiện thực hóa các khả năng như kiểm soát quyền hạn và truy xuất kiểm toán.
- Hệ thống KMS được chia thành ba modules cốt lõi:
    - Enclave: Root bảo mật của toàn hệ thống, chịu trách nhiệm chính về root key của hệ thống lưu trữ bảo mật và chỉ mở quyền truy cập vào các phân hệ chức năng cần thiết trong hệ thống.
    - Service layer: Phần thực hiện các chức năng chính của hệ thống, cung cấp dịch vụ quản lý khóa và mã hóa dữ liệu cho người dùng và ứng dụng KMS, đây cũng là phần KMS liên quan mật thiết nhất đến logic nghiệp vụ.
    - Access layer: Cung cấp khả năng truy cập dịch vụ cho các hệ thống ứng dụng và KMS hỗ trợ tích hợp với chi phí thấp hơn bằng cách cung cấp SDK đa ngôn ngữ và đa khung.
- Từ 3 modules cốt lõi trên, KMS được chia thành 6 modules chức năng:
    - Application management: người quản lý ứng dụng thường là người lãnh đạo, mỗi ứng dụng được gán một khóa duy nhất gọi là masterkey.
    - Key management: quản trị viên có thể tạo, sử dụng và hủy khóa và các khóa này được mã hóa bởi masterkey.
    - Service management: một ứng dụng được hỗ trợ bởi nhiều dịch vụ, do đó, dịch vụ không chịu trách nhiệm quản lý khóa, chỉ yêu cầu quyền truy cập vào khóa từ ứng dụng. Quản trị viên dịch vụ thường là các nhà phát triển. KMS hỗ trợ mã hóa cục bộ để phân phối khóa và mã hóa từ xa, do đó, quản trị viên ứng dụng KMS quyết định cách sử dụng khóa.
    - Approval management: ứng dụng, khóa và dịch vụ. Theo các kịch bản khác nhau, các chức năng phê duyệt đa chiều, đa cấp và có thể mở rộng được thực hiện
    - Audit management: KMS ghi lại từng hoạt động của người dùng để đảm bảo rằng các hoạt động rủi ro của người dùng có thể được theo dõi.
    - Open interface: KMS quản lý khóa dựa trên con người và truy cập khóa dựa trên dịch vụ

# Cloud KMS design pillars

- Customer control: cho phép kiểm soát các key của phần mềm và phần cứng để ký và mã hóa, hỗ trợ bring-your-own-key (BYOK) và hold-your-own-key (HYOK)
    - BYOK và CYOK
        
        ### Bring Your Own Key
        
        - BYOK cho phép end-user tạo, sao lưu và gửi các khóa mã hóa của riêng mình lên đám mây một cách độc lập.
        - BYOK sẽ mất quyền kiểm soát các khóa mã hóa của mình sau khi chúng được tải lên cloud provider.
        - Nếu một khóa bị mất hoặc xảy ra lỗi, dữ liệu không thể được giải mã, điều này có thể dẫn đến bế tắc.
        - Nếu một khóa bị đánh cắp, toàn bộ hoạt động bảo mật sẽ gặp nguy hiểm.
        - Nếu khóa bị mất hoặc bị đánh cắp, có rất ít việc có thể làm được vì ban đầu service provider được miễn trừ trách nhiệm pháp lý của họ đối với khóa.
        - Tổ chức cần thận trọng trong việc duy trì các bản sao lưu và tuân thủ các hoạt động của họ theo các biện pháp bảo mật cao.
        
        ### Hold Your Own Key
        
        - End-user có toàn quyền kiểm soát và quyền sở hữu với các key của họ.
        - Dữ liệu được mã hóa trước khi gửi lên đám mây.
        - Dữ liệu sẽ không được giải mã cho đến khi nó trở về với end-user.
        - HYOK đảm bảo rằng dữ liệu nhạy cảm luôn được mã hóa trong cloud.
        - Các key của end-user không bao giờ bị lộ.
        - End-user giữ quyền sở hữu vật lý và kiểm soát logic với các key mã hóa của mình.
        - HYOK cho phép thu hồi quyền truy cập ngay lập tức bằng cách hủy kích hoạt URL của key
        - Dữ liệu được liên kết với khóa đã hủy kích hoạt sẽ ngay lập tức không thể truy cập được cho đến khi khóa được khôi phục.
        - HYOK lý tưởng cho các tổ chức phải tuân thủ các chính sách tuân thủ và quy định nghiêm ngặt, đặc biệt là các tổ chức tài chính ngân hàng.
- Access control and monitoring: cho phép quản lý các đặc quyền trên từng key và cách chúng được sử dụng
- Regionality: cung cấp khả năng khu vực hóa, service được cấu hình để chỉ tạo, lưu trữ và xử lý khóa trong một vị trí Google Cloud nhất định
- Backup: sao lưu để phòng trường hợp dữ liệu bị hỏng và xác minh rằng dữ liệu có thể được mã hóa thành công, quét và sao lưu các key material và key metadata theo chu kỳ
- Security: bảo vệ khỏi các truy cấp trái phép đến các key và được tích hợp hoàn toàn với Identity and Access Management (IAM) và Cloud Audit Logs

# Source and management options for key materials

Nền tảng Cloud KMS cho phép quản lý các key mã hóa ở trong một dịch vụ cloud trung tâm để sử dụng trực tiếp các tài nguyên và ứng dụng của Google Cloud hoặc từ cloud khác.

![https://cloud.google.com/static/docs/security/key-management-deep-dive/images/kms-source-materials.svg](https://cloud.google.com/static/docs/security/key-management-deep-dive/images/kms-source-materials.svg)

- Default encryption: tất cả dữ liệu được lưu trữ bởi Google sẽ được mã hóa ở storage layer bằng AES-256. Google khởi tạo và quản lý keys cho default encryption, khách hàng không có quyền truy cập đến các keys hay kiểm soát xoay vòng và quản lý khóa. Default encryption keys có thể được chia sẻ giữa các khách hàng
- Cloud KMS with software-generated keys: cho phép người dùng kiểm soát keys được khởi tạo bởi Google, cung cấp tính linh hoạt với việc mã hóa và ký bằng khóa đối xứng hoặc bất đối xứng mà người dùng có thể kiểm soát
- Cloud KMS with hardware-generated keys (Cloud HSM): lưu trữ key với Cloud HSM (Hardware Security Module) backend
- Cloud KMS with key import: để đáp ứng yêu cầu BYOK, khách hàng có thể nhập key tự tạo riêng của mình
- Cloud KMS with external key manager (Cloud EKM): để đáp ứng yêu cầu HYOK, khách hàng có thể tạo và kiểm soát các key trong một trình quản lý key bên ngoài Google Cloud

# Encryption concepts and key management at Google

## Key rings, keys, and key versions

![https://cloud.google.com/static/docs/security/key-management-deep-dive/images/kms-keys-keyrings.svg](https://cloud.google.com/static/docs/security/key-management-deep-dive/images/kms-keys-keyrings.svg)

- Key: là một thực thể được đặt tên đại diện cho khóa mật mã, bao gồm một hoặc nhiều các key versions, cùng với các metadata cho mỗi key. Một key chỉ tồn tại chính xác trên một keyring gắn với một địa điểm cụ thể. Người dùng có thể cho phép hoặc từ chối truy cập đến các keys thông qua IAM (Identity and Access Management). Không thể quản lý truy cập đến một key version.
- Key versions: mỗi version của key bao gồm các thành phần được sử dụng để mã hóa và ký. Mỗi version được biểu diễn bởi 1 số tự nhiên bắt đầu bằng 1. Khi xoay vòng key, một key version mới được tạo với key material mới. Key đối xứng sẽ có một primary version và mặc định nó sẽ dùng để mã hóa. Khi giải mã với key đối xứng, Cloud KMS sẽ tự động xác định key version nào cần dùng để giải mã.
- Key metadata: IAM policies, key type, key size, key state, ...
- Key rings: một nhóm các keys theo một khu vực cụ thể và cho phép người dùng truy cập và quản lý các nhóm keys. Key ring không thể bị xóa sau khi đã được tạo.

⇒ Key rings được dùng để lưu các KEKs

## Default encryption of data at rest

- Algorithm: AES-256 (ngoại trừ một số lượng nhỏ các Persistent Disk được tạo ra từ 2015 dùng AES-128)
- Cryptographic library: Tink
- Module: FIPS 140-2 (BoringCrypto)

### Layers of encryption

- Hình dưới đây thể hiện một vài lớp mã hóa thường được sử dụng để bảo vệ dữ liệu người dùng trong các trung tâm dữ liệu sản phẩm của Google được áp dụng với cả mã hóa hệ thống files phi tập trung hoặc database, mã hóa file lưu trữ đối với người dùng, và cả mã hóa các thiết bị lưu trữ đối với các trung tâm dữ liệu sản phẩm.

![https://cloud.google.com/static/docs/security/encryption/default-encryption/resources/encryption-layers.svg](https://cloud.google.com/static/docs/security/encryption/default-encryption/resources/encryption-layers.svg)

### Encryption at the hardware and infrastructure layer

- Dữ liệu được chia thành các đoạn file con gọi là các data chunks, kích thước của mỗi chunk có thể lên tới vài gigabytes. Mỗi chunk sẽ được mã hóa ở storage level với từng data encryption key (DEK) riêng, hai chunks không dùng chung DEK ngay cả khi chúng được sở hữu bởi cùng một user hay lưu ở cùng machine. (Một chunk trong Datastore, App Engine và Pub/Sub có thể chứa dữ liệu của nhiều khách hàng).
- Nếu một data chunk được cập nhật, nó sẽ được mã hóa bằng 1 key mới thay vì dùng lại key cũ.
- Mỗi data chunk có một định danh độc nhất. Access control lists (ACLs) giúp đảm bảo rằng mỗi chunk sẽ chỉ được giải mã bởi các dịch vụ của Google hoạt động với các vai trò đã được ủy quyền. Các dịch vụ này chỉ được cấp quyền truy cập tại thời điểm đó, điều này giúp ngăn các truy cập trái phép, tăng cường bảo mật và quyền riêng tư.
- Mỗi chunk được phân phối trên các hệ thống lưu trữ và được sao chép ở dạng bản mã để sao lưu và khôi phục. Attacker muốn truy cập vào dữ liệu khách hàng sẽ cần biết 2 điều: tất cả các chunks tương ứng với data muốn biết và tất cả các keys tương ứng với các chunks đó.

![https://cloud.google.com/static/docs/security/encryption/default-encryption/resources/data-upload-chunks.svg](https://cloud.google.com/static/docs/security/encryption/default-encryption/resources/data-upload-chunks.svg)

### Encryption at the storage device layer

- Dữ liệu cũng được mã hóa ở layer thiết bị lưu trữ với AES-256 cho các HDDs và SSDs, sử dụng một device-level riêng khác với key dùng để mã hóa dữ liệu ở storage level.

### Encryption of backups

- Các hệ thống backup đảm bảo dữ liệu được mã hóa trong suốt quá trình backup giúp tránh các trường hợp bị lộ bản rõ.
- Hệ thống backup mã hóa hầu hết các file một cách độc lập với một DEK riêng. DEK này có nguồn gốc từ một key được lưu ở Keystore và được khởi tạo ngẫu nhiên dựa trên từng file trong quá trình backup. Một DEK khác được dùng cho toàn bộ metadata  của các backups và cũng được lưu ở Keystore.

## Key management

### Data encryption keys

- Là key được dùng để mã hóa dữ liệu
- Quản lý DEKs:
    - Được khởi tạo ở local
    - Khi lưu trữ, DEK cần được mã hóa
    - Để truy cập dễ dàng, DEK được lưu ở gần dữ liệu mà nó mã hóa
    - Khởi tạo DEK mỗi lần ghi dữ liệu ⇒ Không cần phải xoay vòng DEKs
    - Không dùng cùng một DEK để mã hóa dữ liệu của 2 người dùng khác nhau
    - Dùng thuật toán mã hóa mạnh như AES-256 với Galois Counter Mode (GCM)

### Key encryption keys

- Là key được dùng để mã hóa DEK
- Quản lý KEKs:
    - Được khởi tạo ở Keystore
    - Lưu trữ KEKs tập trung
    - Đặt các thông tin chi tiết về các DEKs mà KEK mã hóa (Ví dụ: một workload có nhiều DEKs để mã hóa các data chunks của workload đó, thì cần dùng một KEK duy nhất để mã hóa các DEKs đó)
    - Xoay vòng khóa thường xuyên, đặc biệt là sau một sự cố

### Key management

- DEK được lưu gần dữ liệu mà nó mã hóa. DEK được mã hóa bởi KEK, sử dụng kỹ thuật envelope encryption (mã hóa phong bì). Các KEKs sẽ không cụ thể cho từng khách hàng, thay vào đó, một hoặc nhiều KEKs sẽ tương ứng với mỗi service.
    - Envelop encryption
        - Là quá trình mã hóa một khóa bằng một khóa khác.
        - Hình sau thể hiện các levels chính của một phân cấp khóa:
        
        ![https://cloud.google.com/static/kms/images/resources2.png](https://cloud.google.com/static/kms/images/resources2.png)
        
        ### How to encrypt data using envelope encryption
        
        ![https://cloud.google.com/static/kms/images/envelope_encryption_store.svg](https://cloud.google.com/static/kms/images/envelope_encryption_store.svg)
        
        ### How to decrypt data using envelope encryption
        
        ![https://cloud.google.com/static/kms/images/envelope_encryption_retrieve.svg](https://cloud.google.com/static/kms/images/envelope_encryption_retrieve.svg)
        
- Keystore có thể tự động xoay vòng KEKs sau một khoảng thời gian nhất định, sử dụng thư viện mật mã thông dụng của Google để tạo key mới. Key mới sẽ trở thành primary version của key đó, các key cũ sẽ là previous version. Primary key sẽ được dùng để mã hóa và các previous keys được dùng để giải mã.
- KEK được sao lưu cho mục đích khắc phục thảm họa và chúng có thể khôi phục vô thời hạn.
- Việc sử dụng KEKs được quản lý bởi Access Control Lists (ACLs) trong Keystore với mỗi key dựa trên policy của từng key. Chỉ những dịch vụ và người dùng hợp lệ được phép truy cập đến các key này.

### Process for accessing encrypted chunks of data

- Quá trình service truy cập đến các data chunks sẽ diễn ra như sau:
    
    ![https://cloud.google.com/static/docs/security/encryption/default-encryption/resources/process-encrypted-chunks.svg](https://cloud.google.com/static/docs/security/encryption/default-encryption/resources/process-encrypted-chunks.svg)
    
    - Service gọi đến storage system cho dữ liệu cần lấy
    - Storage system xác định những chunks chứa dữ liệu đó (chunk IDs và nơi chúng được lưu)
    - Với mỗi chunk, storage system lấy ra các DEK lưu cùng mỗi chunk và gửi đến Keystore để giải mã
    - Storage system xác minh rằng identified job được phép truy cập đến data chunk dựa vào job identifier và chunk ID. Keystore xác minh rằng storage system được quyền sử dụng KEK tương ứng với dịch vụ đó và giải mã DEK cụ thể
    - Keystore làm một trong những việc sau:
        - Gửi DEK đã giải mã lại cho storage system để giải mã data chunk và gửi dữ liệu về cho dịch vụ
        - Trong một số trường hợp hiếm gặp, Keystore sẽ gửi DEK đã giải mã đến cho dịch vụ. Storage system sẽ gửi data chunk đã mã hóa đến cho dịch vụ và dịch vụ sẽ giải mã để lấy ra dữ liệu
- Quá trình này sẽ khác đối với các thiết bị lưu trữ chuyên dụng, chẳng hạn với SSD cục bộ thì thiết bị sẽ quản lý và bảo vệ DEK cấp độ thiết bị.

### Encryption key hierarchy and root of trust

- Keystore được bảo vệ bởi một root key gọi là keystore master key, có nhiệm vụ mã hóa các KEKs trong Keystore. Keystore master key này là AES-256 key và được lưu ở một KMS khác gọi là Root Keystore. Root keystore này lưu trữ một số lượng rất nhỏ keys - xấp xỉ một chục với mỗi khu vực.
- Mỗi Root Keystore lại có một root key riêng gọi là root keystore master key được lưu trong một cơ sở hạ tầng peer-to-peer gọi là root keystore master key distributor và sao chép các key đó trên toàn cầu. Các root keystore master key distributor này chỉ lưu các key ở trong RAM của cùng một máy chuyên dụng với Root Keystore, và nó sử dụng logging để xác minh việc sử dụng. Một bản (instance) root keystore master key distributor chạy cho mọi bản Root Keystore.
- Khi một bản mới của root keystore master key distributor bắt đầu, nó sẽ được cấu hình với một danh sách các hostname của các bản distributor đang chạy. Bản distributor mới sau đó có thể nhận được các root keystore master key từ các bản đang chạy. Các root keystore master key chỉ tồn tại trong RAM của một số máy được bảo mật đặc biệt.
- Để giải quyết tình huống trong đó tất cả các bản của root keystore master key distributor trong một khu vực khởi động lại đồng thời, root keystore master key cũng được sao lưu trên các thiết bị phần cứng an toàn được lưu trữ trong két vật lý ở các khu vực được bảo mật cao ở nhiều vị trí phân bổ theo địa lý. Bản sao lưu này chỉ được dùng đến nếu tất cả các bản distributor trong một khu vực ngừng hoạt động cùng một lúc. Chỉ có ít hơn 100 nhân viên của Google có thể truy cập vào những két sắt này.

![https://cloud.google.com/static/docs/security/encryption/default-encryption/resources/process-encryption-key-hierarchy.svg](https://cloud.google.com/static/docs/security/encryption/default-encryption/resources/process-encryption-key-hierarchy.svg)

## Software key hierarchy

![https://cloud.google.com/static/docs/security/key-management-deep-dive/images/key-source-hierarchy.svg](https://cloud.google.com/static/docs/security/key-management-deep-dive/images/key-source-hierarchy.svg)

- Dữ liệu được chia thành các đoạn và được mã hóa bằng DEKs
- DEKs được mã hóa bằng KEKs và được lưu cùng với dữ liệu
- KEKs được lưu ở keystore
- Keystore chạy trên nhiều máy trong các trung tâm dữ liệu toàn cầu
- Keystore keys được mã hóa bằng keystore master key được lưu ở Root Keystore
- Root Keystore nhỏ hơn nhiều so với Keystore và được chạy ở các máy chuyên dụng trong các trung tâm dữ liệu
- Root keystore key được mã hóa bằng root keystore master key được lưu ở Root Keystore master key distributor
- Các Root Keystore master key distributor là một cơ sở hạ tầng ngang hàng (peer-to-peer) nằm trong RAM của các máy chuyên dụng toàn cầu, mỗi máy nhận key material của nó từ các máy khác trong khu vực
- Trong trường hợp tất cả các distributor ở một khu vực cùng ngừng hoạt động, một master key được lưu ở thiết bị phần cứng an toàn khác trong một két vật lý ở một địa điểm giới hạn của Google

# Cloud KMS platform overview

![https://cloud.google.com/static/docs/security/key-management-deep-dive/images/cloud-kms-platform.svg](https://cloud.google.com/static/docs/security/key-management-deep-dive/images/cloud-kms-platform.svg)

- Quản trị viên truy cập vào dịch vụ quản lý khóa bằng cách sử dụng Google Cloud Console (gcloud command-line tool) hoặc thông qua các ứng dụng có tích hợp REST hoặc gRPC APIs.
- Các ứng dụng có thể sử dụng các dịch vụ của Google đã enabled CMEK (customer-managed encryption keys). CMEK sau đó sử dụng Cloud KMS API. Cloud KMS API cho phép người dùng có thể sử dụng các khóa phần mềm (Cloud KMS) hoặc phần cứng (Cloud HSM). Cả 2 loại khóa này đều được áp dụng các biện pháp back-up bảo vệ của Google.
- Với Cloud KMS platform, người dùng có thể chọn mức độ bảo vệ khi tạo khóa để xác định cái key backend nào sẽ tạo key và dùng key đó để mã hóa trong tương lai.
- Cloud KMS platform cung cấp 2 backends được hiển thị trong Cloud KMS API dưới dạng:
    - Software Protection Level: phần mềm mức bảo vệ áp dụng cho các key có thể mở khóa bằng module bảo mật phần mềm để thực hiện mã hóa.
    - HSM Protection Level: HSM mức bảo vệ áp dụng cho các key chỉ có thể mở khóa bằng các HSM thực hiện mã hóa với khóa đó.

# Main features of Cloud KMS

- Quản lý khóa: cho phép tạo, quản lý và sử dụng các khóa mã hóa cho các dịch vụ và phần mềm
- Xoay vòng khóa: cung cấp các chính sách xoay vòng khóa giúp cải thiện tính bảo mật
- Tính kết hợp: kết hợp với các dịch vụ khác như Cloud Storage, BigQuery và Compute Engine
- Phân quyền: cho phép kiểm soát ai có thể tạo, quản lý và sử dụng các khóa mã hóa
- Audit Logging: cho phép theo dõi việc sử dụng khóa và đảm bảo tuân thủ chính sách bảo mật