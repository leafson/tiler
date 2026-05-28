# Tiler 瓦片下载工具更新日志

## v0.2.0 (2026-05-27)

### 新增功能

#### 1. 断点续传功能
支持从中断的位置继续下载，避免重复下载已完成的瓦片，大幅提高网络不稳定时的下载效率。

**特性：**
- 自动检测已下载的瓦片文件
- 跳过已存在的瓦片，只下载缺失部分
- 支持文件和MBTiles两种输出格式
- 智能识别现有输出目录

**使用方法：**
```bash
# 新任务下载（不使用断点续传）
./tiler -c conf.toml

# 断点续传 - 指定之前的输出目录
./tiler -c conf.toml -r /path/to/existing/output/directory

# 断点续传 - 指定MBTiles文件
./tiler -c conf.toml -r /path/to/existing/file.mbtiles
```

**示例：**
假设您之前下载了中国区域的瓦片到 `output/china` 目录，中途因网络问题中断：
```bash
# 第一次下载（正常）
./tiler -c conf.toml

# 如果中断后，继续下载缺失的瓦片
./tiler -c conf.toml -r output/google-satelite-z1-16
```

---

#### 2. HTTP/SOCKS5 代理支持
支持通过代理服务器进行瓦片下载，解决网络访问限制和IP限速问题。

**支持的代理类型：**
- SOCKS5 代理（推荐）
- HTTP 代理
- HTTPS 代理

**使用方法：**
```bash
# 使用SOCKS5代理
./tiler -c conf.toml -p socks5://username:password@proxy-server:port

# 使用HTTP代理
./tiler -c conf.toml -p http://proxy-server:8080

# 带认证的代理
./tiler -c conf.toml -p socks5://user:Mima=001@47.254.236.163:1080

# 断点续传 + 代理组合使用
./tiler -c conf.toml -r output/dir -p socks5://proxy:1080
```

**配置建议：**
- 如果使用需要认证的代理，确保密码中的特殊字符进行URL编码
- 代理服务器应保持稳定，频繁切换会影响下载效率

---

#### 3. 日志系统优化
全新的日志输出系统，提供清晰的统计信息和详细的失败记录。

**新特性：**
- **控制台精简输出**：只显示进度条和成功/失败统计
- **失败详细日志**：自动生成独立的失败记录文件
- **实时统计显示**：进度条前缀显示 `[Success: X, Failed: Y]`
- **失败原因记录**：记录每个失败瓦片的Z/X/Y坐标和失败原因

**输出示例：**
```
Task [Success: 1234, Failed: 5] :  12345 / 100000 [====->----------------------------]  12.3% 15m30s
```

**失败日志文件：**
- 位置：`output/{task-id}-failed.log`
- 格式：`[时间戳] Failed tile: z=层级, x=列, y=行, reason: 失败原因`
- 内容示例：
```
[2026-05-27 11:49:52] Failed tile: z=14, x=9889, y=7299, reason: status code: 404
[2026-05-27 11:50:15] Failed tile: z=14, x=9900, y=7331, reason: read error: connection reset
```

**失败处理流程：**
1. 下载完成后查看失败日志文件
2. 根据日志定位失败的瓦片
3. 使用 `-r` 参数进行补全下载
4. 工具会自动跳过已成功下载的瓦片，只补全失败的瓦片

---

### 改进内容

1. **并发控制优化**
   - 保持了原有的多线程下载机制
   - 优化了并发任务的统计同步

2. **状态检查增强**
   - MBTiles模式：查询数据库判断瓦片是否存在
   - 文件模式：检查文件系统中文件是否存在
   - 双重保障确保不会重复下载

3. **错误处理完善**
   - HTTP状态码错误记录
   - 网络连接异常记录
   - 空瓦片数据记录
   - 文件保存失败记录

---

### 命令行参数完整说明

```bash
Usage: tiler [-h] [-c filename] [-r directory] [-p proxy]
```

| 参数 | 说明 | 必填 | 默认值 | 示例 |
|------|------|------|--------|------|
| `-h` | 显示帮助信息 | 否 | - | `./tiler -h` |
| `-c` | 配置文件路径 | 否 | `conf.toml` | `./tiler -c my-config.toml` |
| `-r` | 断点续传的目录或文件 | 否 | 无 | `./tiler -r output/tiles` |
| `-p` | 代理服务器URL | 否 | 无代理 | `./tiler -p socks5://proxy:1080` |

---

### 典型使用场景

#### 场景1：首次下载全国地图瓦片
```bash
./tiler -c conf.toml
```

#### 场景2：下载过程中断线后的续传
```bash
# 重新执行相同的命令，指定输出目录
./tiler -c conf.toml -r output/google-satelite-z1-16.SSAw0PJDg
```

#### 场景3：遇到网络限制使用代理
```bash
# 通过代理服务器下载
./tiler -c conf.toml -p socks5://user:pass@proxy-server:1080
```

#### 场景4：补全失败的瓦片
```bash
# 1. 查看失败日志
cat output/*-failed.log

# 2. 使用断点续传补全
./tiler -c conf.toml -r output/google-satelite-z1-16
```

#### 场景5：代理 + 断点续传组合
```bash
# 使用代理并从中断处继续
./tiler -c conf.toml -r output/dir -p socks5://proxy:1080
```

---

### 注意事项

1. **断点续传注意事项：**
   - 必须使用与原始下载相同的配置文件
   - 输出目录结构必须保持一致
   - 不同缩放级别的瓦片需要分别续传

2. **代理使用注意事项：**
   - 确保代理服务器稳定可用
   - 代理认证信息要正确填写
   - 某些严格的代理可能会影响大量并发请求

3. **失败处理建议：**
   - 定期检查失败日志文件
   - 补全下载前等待一段时间（如网络临时故障）
   - 大量失败时检查网络连接和代理设置

4. **性能优化建议：**
   - 根据网络状况调整 `task.workers` 参数
   - 设置合适的 `task.timedelay` 避免被限速
   - 使用SSD存储提高文件IO性能

---

### 技术细节

#### 断点续传实现原理
- **文件格式**：通过 `{zoom}/{x}/{y}.format` 路径检查文件存在性
- **MBTiles格式**：查询SQLite数据库中 `(zoom_level, tile_column, tile_row)` 唯一索引
- **Y轴翻转**：正确处理TMS和XYZ坐标系的Y轴差异

#### 统计信息同步
- **successCount**：原子计数器记录成功下载的瓦片数
- **failedCount**：原子计数器记录失败的瓦片数
- **实时更新**：每次进度递增时同步显示统计信息

#### 日志文件管理
- **自动创建**：任务开始时自动创建失败日志文件
- **追加模式**：支持多次补全下载的日志追加
- **自动关闭**：任务完成后自动关闭文件句柄

---

### 版本兼容性

- 向后兼容 v0.1.0 的所有功能
- 新的 `-r` 和 `-p` 参数为可选参数
- 不影响现有的下载任务和配置文件

---

### 反馈与支持

如有问题或建议，请查阅：
- 失败日志文件位置：`output/{task-id}-failed.log`
- 配置文件说明：参见 `conf.toml`
- 支持的地图源：天地图、Google Maps、OpenStreetMap、Mapbox等
