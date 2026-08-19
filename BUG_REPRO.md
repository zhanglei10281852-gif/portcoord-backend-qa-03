# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

同一个幂等键重复提交到港申报时，第二次响应丢失了第一次生成的申报编号。请修复缓存响应的跨层传递，重复请求必须返回完整且一致的结果。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/portcoord-backend-qa-03
- 仓库地址：https://github.com/zhanglei10281852-gif/portcoord-backend-qa-03.git
- parent SHA：a5a2cfba78d53b47710ced29bb250f44e7fe2160

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/portcoord-backend-qa-03.git bug-repro
cd bug-repro
git checkout --detach a5a2cfba78d53b47710ced29bb250f44e7fe2160
go test ./internal/declaration -run "^TestDeclaration_Submit_IdempotentDuplicate$" -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/declaration -run "^TestDeclaration_Submit_IdempotentDuplicate$" -count=1
--- FAIL: TestDeclaration_Submit_IdempotentDuplicate (0.01s)
    declaration_test.go:68: idempotent submit should return same ID: 4d83c202-7823-43e6-ba30-0c48b20c1a6d vs 
FAIL
FAIL	portcoord/internal/declaration	0.018s
FAIL

```

stderr：

```text
warning: internal/declaration/declaration_test.go has type 100755, expected 100644
warning: internal/declaration/test_helpers_test.go has type 100755, expected 100644
warning: internal/declaration/declaration_test.go has type 100755, expected 100644
warning: internal/declaration/test_helpers_test.go has type 100755, expected 100644

```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/declaration -run "^TestDeclaration_Submit_IdempotentDuplicate$" -count=1
--- FAIL: TestDeclaration_Submit_IdempotentDuplicate (0.26s)
    declaration_test.go:68: idempotent submit should return same ID: f1015f11-5f22-46ed-a827-0fd0326c6219 vs 
FAIL
FAIL	portcoord/internal/declaration	0.503s
FAIL

```

stderr：

```text
warning: internal/declaration/declaration_test.go has type 100755, expected 100644
warning: internal/declaration/test_helpers_test.go has type 100755, expected 100644
warning: internal/declaration/declaration_test.go has type 100755, expected 100644
warning: internal/declaration/test_helpers_test.go has type 100755, expected 100644

```

## 通过条件

在题面触发条件下，公开行为必须恢复且原始异常不再出现；定向命令 go test ./internal/declaration -run ^TestDeclaration_Submit_IdempotentDuplicate$ -count=1、相关包测试、全量测试、race、vet 和 build 必须通过；不得删除或跳过测试，也不得绕过目标逻辑。
