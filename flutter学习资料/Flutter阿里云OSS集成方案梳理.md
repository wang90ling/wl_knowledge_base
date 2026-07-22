# Flutter 阿里云 OSS 集成方案梳理

## 一、方案对比

| 方案 | 原理 | 优点 | 缺点 | 推荐度 |
|------|------|------|------|--------|
| **flutter_oss_aliyun 插件** | 封装好的阿里云 OSS SDK | 开箱即用、功能完善 | 第三方维护、更新不及时 | ⭐⭐⭐⭐ |
| **HTTP 直传（Dio/http）** | 通过临时凭证直接调用 OSS API | 灵活、无第三方依赖 | 需要自己处理签名和各种场景 | ⭐⭐⭐⭐⭐ |
| **Platform Channel 调原生** | 通过 MethodChannel 调用 Android/iOS 原生 SDK | 与原生保持一致 | 需要维护原生代码、跨平台复杂 | ⭐⭐⭐ |

## 二、推荐方案：HTTP 直传 + STS 临时凭证

### 架构设计

```
┌─────────────────────────────────────────────────────────────────┐
│                        Flutter App                             │
│  ┌─────────────────┐   ┌─────────────────┐                    │
│  │  UI 层          │   │  媒体选择层     │                    │
│  │  (发布动态等)   │   │  (image_picker) │                    │
│  └────────┬────────┘   └────────┬────────┘                    │
│           │                     │                             │
│           ▼                     ▼                             │
│  ┌───────────────────────────────────────────────────┐        │
│  │              OssService 服务层                     │        │
│  │  ┌─────────────────┐   ┌─────────────────┐        │        │
│  │  │ getStsToken()   │   │ uploadFile()    │        │        │
│  │  │ 获取临时凭证     │   │ 上传文件        │        │        │
│  │  └────────┬────────┘   └────────┬────────┘        │        │
│  └───────────┼──────────────────────┼─────────────────┘        │
│              │                      │                          │
└──────────────┼──────────────────────┼──────────────────────────┘
               │                      │
               ▼                      ▼
    ┌──────────────────┐    ┌──────────────────┐
    │   后端服务器      │    │   阿里云 OSS      │
    │   /api/sts/token │    │   PutObject      │
    └──────────────────┘    └──────────────────┘
```

## 三、详细实施步骤

### 第一步：添加依赖

```yaml
dependencies:
  flutter:
    sdk: flutter
  http: ^1.2.2
  image_picker: ^1.1.2
  crypto: ^3.0.5  # 用于签名计算
  uuid: ^4.4.0     # 生成唯一文件名
  dio: ^5.4.0      # 可选，支持断点续传
```

### 第二步：创建 OSS 配置类

```dart
// lib/core/services/oss_config.dart
class OssConfig {
  static const String endpoint = 'oss-cn-beijing.aliyuncs.com';
  static const String bucket = 'your-bucket-name';
  static const String stsApi = '${NetworkEndpoints.baseUrl}api/sts/token';
  
  static String get uploadUrl => 'https://$bucket.$endpoint';
}
```

### 第三步：创建 STS 凭证模型

```dart
// lib/models/sts_token_response.dart
class StsTokenResponse {
  final String accessKeyId;
  final String accessKeySecret;
  final String securityToken;
  final String expiration;

  StsTokenResponse({
    required this.accessKeyId,
    required this.accessKeySecret,
    required this.securityToken,
    required this.expiration,
  });

  factory StsTokenResponse.fromJson(Map<String, dynamic> json) {
    return StsTokenResponse(
      accessKeyId: json['AccessKeyId'] ?? '',
      accessKeySecret: json['AccessKeySecret'] ?? '',
      securityToken: json['SecurityToken'] ?? '',
      expiration: json['Expiration'] ?? '',
    );
  }
}
```

### 第四步：创建 OSS 服务类

```dart
// lib/core/services/oss_service.dart
import 'dart:convert';
import 'dart:io';
import 'dart:typed_data';

import 'package:crypto/crypto.dart';
import 'package:http/http.dart' as http;
import 'package:image_picker/image_picker.dart';
import 'package:uuid/uuid.dart';

import '../network/api_service.dart';
import '../helpers/app_logger.dart';
import '../models/sts_token_response.dart';
import 'oss_config.dart';

class OssService {
  OssService({ApiService? apiService}) : _apiService = apiService ?? ApiService();

  final ApiService _apiService;
  StsTokenResponse? _cachedToken;

  /// 获取 STS 临时凭证
  Future<StsTokenResponse> _getStsToken() async {
    // 如果缓存的 token 还没过期，直接返回
    if (_cachedToken != null) {
      final expiration = DateTime.parse(_cachedToken!.expiration);
      if (expiration.isAfter(DateTime.now().add(const Duration(minutes: 5)))) {
        return _cachedToken!;
      }
    }

    final uri = Uri.parse(OssConfig.stsApi);
    final response = await http.get(uri);
    final data = json.decode(response.body);
    
    _cachedToken = StsTokenResponse.fromJson(data);
    return _cachedToken!;
  }

  /// 生成阿里云 OSS 签名
  Map<String, String> _generateSignature(
    String method,
    String key,
    StsTokenResponse token,
    DateTime date,
  ) {
    final gmtDate = date.toUtc().toIso8601String().replaceAll('Z', '');
    
    // 构建待签名字符串
    final canonicalizedResource = '/${OssConfig.bucket}/$key';
    final stringToSign = '$method\n\n\n$gmtDate\n$canonicalizedResource';
    
    // HMAC-SHA1 签名
    final keyBytes = utf8.encode(token.accessKeySecret);
    final dataBytes = utf8.encode(stringToSign);
    final hmac = Hmac(sha1, keyBytes);
    final digest = hmac.convert(dataBytes);
    final signature = base64.encode(digest.bytes);

    return {
      'Host': '${OssConfig.bucket}.${OssConfig.endpoint}',
      'Date': gmtDate,
      'Authorization': 'OSS ${token.accessKeyId}:$signature',
      'x-oss-security-token': token.securityToken,
      'Content-Type': 'application/octet-stream',
    };
  }

  /// 上传文件到 OSS
  Future<String> uploadFile({
    required XFile file,
    String? customKey,
    required String fileType, // 'image' or 'video'
  }) async {
    final token = await _getStsToken();
    final uuid = const Uuid().v4();
    
    // 生成文件名
    final extension = file.path.split('.').last;
    final timestamp = DateTime.now().millisecondsSinceEpoch;
    final key = customKey ?? '$fileType/$timestamp/$uuid.$extension';
    
    final fileBytes = await file.readAsBytes();
    final uri = Uri.parse('${OssConfig.uploadUrl}/$key');
    
    final headers = _generateSignature(
      'PUT',
      key,
      token,
      DateTime.now(),
    );

    final response = await http.put(
      uri,
      headers: headers,
      body: fileBytes,
    );

    if (response.statusCode == 200) {
      final url = 'https://${OssConfig.bucket}.${OssConfig.endpoint}/$key';
      AppLogger.info('文件上传成功: $url', tag: 'OssService');
      return url;
    } else {
      throw Exception('上传失败: ${response.statusCode}');
    }
  }

  /// 批量上传文件
  Future<List<String>> uploadFiles({
    required List<XFile> files,
    required String fileType,
    void Function(int, int)? onProgress,
  }) async {
    final urls = <String>[];
    
    for (var i = 0; i < files.length; i++) {
      final url = await uploadFile(
        file: files[i],
        fileType: fileType,
      );
      urls.add(url);
      onProgress?.call(i + 1, files.length);
    }
    
    return urls;
  }
}
```

### 第五步：在网络层集成

```dart
// lib/core/network/api_service.dart
class ApiService {
  // ... 现有代码 ...
  
  // 添加 OSS 服务实例
  OssService get ossService => OssService(apiService: this);
}
```

### 第六步：在页面中使用

```dart
// lib/pages/circle/circle_page.dart
class _CirclePageState extends State<CirclePage> {
  final OssService _ossService = OssService();
  final ImagePicker _imagePicker = ImagePicker();
  
  Future<void> _uploadMedia() async {
    // 选择图片
    final images = await _imagePicker.pickMultiImage();
    if (images.isEmpty) return;
    
    try {
      // 上传到 OSS
      final urls = await _ossService.uploadFiles(
        files: images,
        fileType: 'image',
        onProgress: (current, total) {
          print('上传进度: $current/$total');
        },
      );
      
      // 上传成功后，将 URL 传给后端保存
      await _apiService.savePost(
        content: _contentController.text,
        imageUrls: urls,
      );
    } catch (e) {
      AppLogger.error('上传失败: $e', tag: 'CirclePage');
    }
  }
}
```

## 四、安全注意事项

### 1. 凭证安全

```
绝对禁止：
❌ 硬编码 AccessKeyId 和 AccessKeySecret
❌ 将凭证存储在客户端代码中

正确做法：
✅ 通过后端 STS 接口获取临时凭证
✅ 凭证有效期设置合理（建议 1-2 小时）
✅ 使用 RAM 子账号 + 最小权限原则
```

### 2. 后端 STS 接口示例（伪代码）

```java
// Java 后端示例 - 获取 STS 临时凭证
@GetMapping("/api/sts/token")
public Map<String, Object> getStsToken() {
    // 1. 创建 STS Client
    // 2. 调用 AssumeRole 获取临时凭证
    // 3. 返回 AccessKeyId, AccessKeySecret, SecurityToken, Expiration
    // 4. 注意：需要配置 RAM 角色和权限策略
}
```

## 五、与鸿蒙端保持一致

```
Flutter 端与鸿蒙端对比：
─────────────────────────────────────────────────
| 模块        | Flutter 端              | 鸿蒙端              |
|─────────────|─────────────────────────|─────────────────────|
| 凭证获取    | HTTP 请求后端 STS 接口   | HTTP 请求后端 STS 接口 |
| 签名方式    | HMAC-SHA1               | HMAC-SHA1           |
| 上传方式    | PUT Object              | PUT Object          |
| 存储路径    | image/{timestamp}/{uuid} | image/{timestamp}/{uuid} |
| 文件名      | uuid.extension          | uuid.extension      |
```

### 关键保持一致的点

1. **后端 STS 接口** - 统一使用同一个接口
2. **文件命名规则** - 保持一致的目录结构和命名格式
3. **Bucket 和 Endpoint** - 使用相同的配置
4. **签名算法** - 都使用 HMAC-SHA1

## 六、进阶优化

### 1. 分片上传（大文件）

```dart
// 对于视频等大文件，建议使用分片上传
Future<String> uploadLargeFile({
  required XFile file,
  int partSize = 1024 * 1024 * 4, // 4MB 分片
}) async {
  // 1. 初始化分片上传 - InitiateMultipartUpload
  // 2. 分片上传 - UploadPart
  // 3. 合并分片 - CompleteMultipartUpload
}
```

### 2. 上传进度监听

```dart
// 使用 dio 实现进度监听
final dio = Dio();
final response = await dio.put(
  url,
  data: fileBytes,
  onSendProgress: (sent, total) {
    final progress = (sent / total * 100).toStringAsFixed(1);
    print('上传进度: $progress%');
  },
);
```

### 3. 文件压缩

```dart
// 图片压缩
import 'package:image/image.dart' as img;

Future<XFile> compressImage(XFile file, {int quality = 80}) async {
  final bytes = await file.readAsBytes();
  final image = img.decodeImage(bytes);
  
  final compressed = img.encodeJpg(
    image!,
    quality: quality,
  );
  
  final tempFile = await File('${file.path}_compressed.jpg').create();
  await tempFile.writeAsBytes(compressed);
  
  return XFile(tempFile.path);
}
```

## 七、集成清单

```
集成步骤检查清单：
☐ 1. 后端部署 STS 临时凭证服务
☐ 2. 配置阿里云 OSS Bucket（权限、跨域）
☐ 3. 添加依赖（crypto, uuid, dio）
☐ 4. 创建 OssConfig 配置类
☐ 5. 创建 StsTokenResponse 模型
☐ 6. 创建 OssService 服务类
☐ 7. 在页面中集成上传逻辑
☐ 8. 测试上传功能
☐ 9. 添加错误处理和进度显示
```

## 八、项目代码参考

### 现有网络层结构

```
lib/core/
├── helpers/
│   ├── app_logger.dart          # 日志工具
│   └── auth_storage.dart        # 认证存储
├── network/
│   ├── api_service.dart         # 主 API 服务
│   ├── circle_api_service.dart  # 圈子 API 服务
│   └── network_client.dart      # 网络客户端
└── services/
    ├── permission_manager.dart   # 权限管理
    └── permission_service.dart   # 权限服务
```

### 新增文件结构

```
lib/
├── core/
│   └── services/
│       └── oss_service.dart      # OSS 服务（新增）
├── models/
│   └── sts_token_response.dart   # STS 凭证模型（新增）
└── core/
    └── services/
        └── oss_config.dart       # OSS 配置（新增）
```

## 九、总结

**推荐方案：HTTP 直传 + STS 临时凭证**

这是最灵活、最安全的方案，与鸿蒙端保持一致的后端接口，无需维护原生代码。

### 核心流程

1. Flutter 调用后端 STS 接口获取临时凭证
2. 使用临时凭证对上传请求签名
3. 直接向阿里云 OSS 发送 PUT 请求上传文件
4. 获取返回的文件 URL，传给业务后端保存

### 下一步建议

1. 先确认后端 STS 接口是否已就绪
2. 确认 Bucket 名称、Endpoint 等配置
3. 按照集成清单逐步实现代码
4. 测试上传功能并添加错误处理