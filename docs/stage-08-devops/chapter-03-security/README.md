# 8.3 镜像安全

## 目标

- 理解容器镜像的安全风险。
- 掌握 Trivy 等安全扫描工具的使用。
- 了解镜像签名和验证。

## 安全扫描

### Trivy 扫描

```bash
# 扫描镜像
trivy image php:8.2-fpm

# 扫描 Dockerfile
trivy fs .

# 生成报告
trivy image --format json -o report.json php:8.2-fpm
```

## 镜像签名

### cosign 签名

```bash
# 生成密钥对
cosign generate-key-pair

# 签名镜像
cosign sign --key cosign.key myimage:tag

# 验证签名
cosign verify --key cosign.pub myimage:tag
```

## 最佳实践

1. 定期扫描镜像漏洞。
2. 使用最小权限运行容器。
3. 及时更新基础镜像。
4. 签名生产镜像。

## 练习

1. 配置 Trivy 扫描 CI 流程。

2. 实现镜像签名和验证流程。

3. 创建安全扫描报告系统。

4. 设计镜像安全策略。
