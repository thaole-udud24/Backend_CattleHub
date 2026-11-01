# Quy tắc Git & Commit

Tài liệu này quy định cách làm việc với Git để mã nguồn dễ quản lý, dễ review, **không push file nhạy cảm** và **không ai push thẳng lên nhánh chính**.

## 1. Nguyên tắc chung

- **Luồng code bắt buộc: branch riêng → `development` → `main`.** Code phải lên `development` và được kiểm tra xong thì mới được đưa lên `main`. Không được bỏ qua `development`.
- **Dev chỉ tạo PR vào `development`.** PR từ `development` vào `main` (release) do người phụ trách release/tech lead thực hiện.
- **Không push trực tiếp lên `development` và `main`.** Mọi thay đổi đều đi qua Pull Request (PR).
- Mỗi PR chỉ làm **một việc** (một feature, một bugfix, một refactor).
- Code phải **build và test pass** trước khi tạo PR.
- Mỗi PR cần **tối thiểu 1 reviewer approve** mới được merge vào `development`.
- **Tuyệt đối không commit file nhạy cảm** (xem mục 3).

## 2. Mô hình branch

```
feature/*  ─┐
fix/*      ─┼──► PR ──► development ──► PR (release) ──► main
refactor/* ─┤          (dev tạo PR)     (tech lead)      (production)
chore/*    ─┘
```

| Branch | Mục đích | Quy tắc |
|---|---|---|
| `development` | Nhánh tích hợp, đích của mọi PR từ dev; test/staging | Protected, chỉ merge qua PR |
| `main` | Code ổn định/production | Protected, chỉ nhận PR từ `development` |
| `feature/*` | Phát triển tính năng mới | Tách từ `development` |
| `fix/*` | Sửa lỗi | Tách từ `development` |
| `refactor/*` | Tái cấu trúc, không đổi behavior | Tách từ `development` |
| `chore/*` | Cấu hình, CI, dependency, tài liệu | Tách từ `development` |

### Quy ước đặt tên branch

```
<loại>/<ticket-id>-<mô-tả-ngắn-kebab-case>
```

Ví dụ:

```
feature/PROJ-123-add-user-login
fix/PROJ-456-null-pointer-order-service
refactor/PROJ-321-split-auth-module
chore/update-eslint-config
```

Quy tắc: chữ thường, dùng dấu gạch ngang, ngắn gọn, có mã ticket nếu có.

## 3. Không push file nhạy cảm và file rác

### Tuyệt đối không commit

- File môi trường: `.env`, `.env.local`, `.env.production`, `.env.*` (trừ `.env.example`)
- Secret: API key, token, mật khẩu, private key (`*.pem`, `*.key`), file credentials, service account JSON
- Cấu hình chứa thông tin thật: `application-prod.yml` có password, `appsettings.Production.json`, `config/secrets.*`
- Dữ liệu thật của khách hàng, dump database (`*.sql`, `*.dump`, `*.bak`)
- File sinh ra khi build/chạy: `node_modules/`, `dist/`, `build/`, `target/`, `bin/`, `obj/`, `__pycache__/`, `*.log`, `coverage/`
- File của IDE/OS: `.idea/`, `.vscode/` (trừ file chia sẻ chung), `.DS_Store`, `Thumbs.db`

### Cách làm đúng

1. Khai báo trong `.gitignore` ngay từ đầu dự án:

```gitignore
# Env & secrets
.env
.env.*
!.env.example
*.pem
*.key
*credentials*.json
secrets/

# Build & dependency
node_modules/
dist/
build/
target/
bin/
obj/
__pycache__/
coverage/

# Logs & data
*.log
*.sql
*.dump
*.bak

# IDE / OS
.idea/
.vscode/
.DS_Store
Thumbs.db
```

2. Tạo file **`.env.example`** chỉ chứa tên biến và giá trị giả, commit file này để người khác biết cần cấu hình gì:

```env
DB_HOST=localhost
DB_PORT=5432
DB_USER=your_user
DB_PASSWORD=your_password
JWT_SECRET=change_me
```

3. Secret thật chia sẻ qua kênh an toàn (password manager, vault, secret của CI/CD), **không gửi qua chat/email**.

4. Trước khi commit, luôn kiểm tra:

```bash
git status
git diff --staged
```

Không dùng `git add .` bừa bãi; ưu tiên `git add <file>` hoặc `git add -p`.

### Tự động chặn secret (khuyến nghị)

Dùng **gitleaks** làm pre-commit hook:

```bash
# cài gitleaks (ví dụ macOS)
brew install gitleaks

# .git/hooks/pre-commit
#!/bin/sh
gitleaks protect --staged --redact || {
  echo "❌ Phát hiện secret trong code đang commit. Hãy gỡ ra trước."
  exit 1
}
```

```bash
chmod +x .git/hooks/pre-commit
```

Nên chạy thêm gitleaks trong CI để chặn PR có secret.

### Nếu lỡ commit/push secret

1. **Đổi (rotate) secret ngay lập tức**, coi như đã bị lộ. Xóa file khỏi commit mới nhất là chưa đủ vì vẫn còn trong lịch sử Git.
2. Báo cho team lead.
3. Gỡ khỏi lịch sử bằng `git filter-repo` hoặc BFG Repo-Cleaner (cần phối hợp cả team vì phải force push và mọi người clone lại).
4. Nếu file chỉ mới `git add` mà chưa commit: `git restore --staged .env` và thêm vào `.gitignore`.
5. Nếu file đã bị track từ trước: `git rm --cached .env` rồi commit.

## 4. Quy tắc commit message (Conventional Commits)

### Định dạng

```
<type>(<scope>): <subject>

[body - tùy chọn]

[footer - tùy chọn]
```

### Các `type`

| Type | Ý nghĩa |
|---|---|
| `feat` | Thêm tính năng mới |
| `fix` | Sửa lỗi |
| `refactor` | Tái cấu trúc code, không thêm tính năng hay sửa lỗi |
| `docs` | Thay đổi tài liệu |
| `style` | Format, dấu chấm phẩy, khoảng trắng (không đổi logic) |
| `test` | Thêm/sửa test |
| `chore` | Việc vặt: build, dependency, cấu hình |
| `perf` | Cải thiện hiệu năng |
| `ci` | Thay đổi CI/CD |
| `revert` | Hoàn tác commit trước đó |

### Quy tắc viết subject

- Tối đa **72 ký tự**.
- Dùng **thể mệnh lệnh**, bắt đầu bằng động từ: `add`, `fix`, `update`, `remove`.
- **Không viết hoa** chữ đầu, **không có dấu chấm** ở cuối.
- `scope` là module/domain bị ảnh hưởng (ví dụ `auth`, `order`, `user`, `db`).
- **Mô tả phải rõ ràng, cụ thể**: người đọc `git log` phải hiểu được commit làm **gì** và ở **đâu** mà không cần mở code. Nêu rõ đối tượng bị tác động (endpoint, hàm, màn hình, bảng DB...) và hành động cụ thể.
- Cấm subject mơ hồ, chung chung: `update code`, `fix bug`, `sửa lỗi`, `cập nhật`, `WIP`, `abc`, `final`...
- Commit thay đổi logic, sửa lỗi hoặc ảnh hưởng nhiều file **bắt buộc có body**: giải thích **vì sao** thay đổi và **thay đổi gì** (với fix lỗi: nêu nguyên nhân và cách khắc phục), wrap ở 100 ký tự.
- Footer dùng để tham chiếu ticket hoặc đánh dấu breaking change.

### Ví dụ tốt

```
feat(auth): add JWT refresh token endpoint

fix(order): prevent duplicate order on double submit

refactor(user): extract validation into separate service

docs(readme): add local setup instructions

chore(deps): upgrade typeorm to 0.3.20
```

Có body và footer:

```
fix(payment): handle timeout when calling gateway

Gateway đôi khi trả về chậm hơn 10s khiến request bị treo.
Thêm timeout 5s và retry tối đa 2 lần với backoff.

Closes PROJ-789
```

Breaking change:

```
feat(api)!: change response format of /users

BREAKING CHANGE: field `fullName` được tách thành `firstName` và `lastName`.
```

### Ví dụ xấu (tránh)

```
update code
fix bug
abc
WIP
final final v2
```

Mơ hồ vs rõ ràng:

```
❌ fix(order): fix bug
✅ fix(order): return 409 instead of 500 when order code already exists

❌ feat(user): update user
✅ feat(user): add PATCH /users/:id to change email and phone number
```

## 5. Quy tắc commit

- **Commit nhỏ, thường xuyên**, mỗi commit là một thay đổi có ý nghĩa và build được.
- Không gộp nhiều việc không liên quan vào một commit.
- Không commit code đang lỗi, code debug (`console.log`, `print`), `TODO` tạm hoặc code bị comment bỏ.
- Trước khi commit: chạy lint, format, test và kiểm tra `git diff --staged` để chắc không lẫn file nhạy cảm.
- Dự án đã cài husky + lint-staged: hook pre-commit tự chạy `eslint --fix` trên file `.js` đã stage, lỗi lint sẽ chặn commit.
- Dùng `git commit --amend` hoặc `git rebase -i` để dọn commit **trên branch của mình** trước khi tạo PR (không làm trên branch đã chia sẻ).

## 6. Quy trình làm việc

```bash
# 1. Cập nhật code mới nhất
git checkout development
git pull origin development

# 2. Tạo branch mới từ development
git checkout -b feature/PROJ-123-add-user-login

# 3. Code, commit nhỏ và thường xuyên
git add <file>
git commit -m "feat(auth): add login endpoint"

# 4. Đồng bộ với development trước khi tạo PR
git fetch origin
git rebase origin/development      # hoặc merge, theo quy ước team

# 5. Push branch của bạn (KHÔNG push development/main)
git push -u origin feature/PROJ-123-add-user-login

# 6. Tạo Pull Request: base = development
```

Sau khi PR được merge, xóa branch:

```bash
git branch -d feature/PROJ-123-add-user-login
git push origin --delete feature/PROJ-123-add-user-login
```

## 7. Quy trình release: `development` → `main`

Chỉ thực hiện khi `development` đã ổn định (CI xanh, đã test xong trên môi trường dev/staging).

1. Người phụ trách release tạo PR: **base = `main`, compare = `development`**, tiêu đề dạng `release: v1.2.0`.
2. Mô tả PR liệt kê các thay đổi chính (changelog) và các bước deploy/migration nếu có.
3. CI xanh và có tối thiểu **1 approve** (khuyến nghị 2) từ tech lead/reviewer.
4. Dùng **Merge commit** (không squash) để giữ lịch sử các PR trên `main`.
5. Sau khi merge, gắn tag phiên bản:

```bash
git checkout main
git pull origin main
git tag -a v1.2.0 -m "release: v1.2.0"
git push origin v1.2.0
```

6. Nếu `main` có thay đổi mà `development` chưa có (ví dụ hotfix), phải merge ngược `main` về `development` để hai nhánh luôn đồng bộ.

### Hotfix khẩn cấp trên production

Vẫn phải qua `development`, không push thẳng `main`:

1. Tạo `fix/PROJ-789-ten-loi` từ `development`, sửa lỗi, tạo PR vào `development`.
2. Review nhanh và merge.
3. Tạo PR release `development` → `main` ngay sau đó.

Chỉ khi `development` đang chứa code chưa sẵn sàng release, tech lead mới được quyết định tạo `hotfix/*` từ `main`, PR vào `main`, rồi **bắt buộc merge ngược về `development`**.

## 8. Quy tắc Pull Request

- **Base branch của PR do dev tạo luôn là `development`.** Chỉ PR release mới có base là `main`.
- Tiêu đề PR theo định dạng commit: `feat(auth): add login endpoint`.

### Mẫu mô tả PR

```markdown
## Mô tả
Thay đổi gì và vì sao.

## Loại thay đổi
- [ ] feat
- [ ] fix
- [ ] refactor
- [ ] docs / chore

## Ticket liên quan
PROJ-123

## Cách test
1. ...
2. ...

## Checklist
- [ ] Đã tự review code
- [ ] Build và test pass
- [ ] Đã thêm/cập nhật test
- [ ] Đã cập nhật tài liệu (nếu cần)
- [ ] KHÔNG có .env, secret, key, dữ liệu thật trong thay đổi
- [ ] Không có code debug hoặc file build
```

### Yêu cầu khi merge

- CI xanh (build, lint, test, quét secret).
- Tối thiểu **1 approve** (khuyến nghị 2 với thay đổi lớn).
- Đã giải quyết hết comment của reviewer.
- Không còn conflict với `development`.
- Ưu tiên **Squash and merge** để lịch sử gọn; tiêu đề squash theo Conventional Commits.
- PR nên nhỏ, dưới khoảng **400 dòng** thay đổi nếu có thể.
- Không tự merge PR của mình khi chưa có người review.

## 9. Bảo vệ branch (cấu hình repo)

Admin tạo **Ruleset** (Settings → Rules → Rulesets) cho `development` và `main`, trạng thái **Active**:

- Restrict deletions
- Block force pushes
- Require a pull request before merging, với:
  - Required approvals >= 1
  - Dismiss stale approvals when new commits are pushed
  - Require conversation resolution before merging
  - Allowed merge methods: `main` chỉ **Merge**, `development` chỉ **Squash**
- Require status checks to pass (build, lint, test, quét secret) khi repo đã có CI
- Bypass list: chỉ chủ repo/tech lead; các thành viên khác chỉ có quyền Write nên không push thẳng được

## 10. Chặn push nhầm ở máy local (tùy chọn)

Tạo `.git/hooks/pre-push`:

```bash
#!/bin/sh
branch=$(git rev-parse --abbrev-ref HEAD)
if [ "$branch" = "main" ] || [ "$branch" = "development" ]; then
  echo "❌ Không được push trực tiếp lên $branch. Hãy tạo branch riêng và mở PR vào development."
  exit 1
fi
```

```bash
chmod +x .git/hooks/pre-push
```

Kiểm tra commit message bằng **commitlint + husky** (nếu dự án dùng Node):

```bash
npm i -D @commitlint/cli @commitlint/config-conventional husky
echo "module.exports = { extends: ['@commitlint/config-conventional'] };" > commitlint.config.js
npx husky init
echo "npx --no -- commitlint --edit \$1" > .husky/commit-msg
```

> Hook local chỉ là lớp bảo vệ phụ, có thể bị bỏ qua. Lớp bảo vệ thật sự là Branch protection ở mục 9.

## 11. Những điều không được làm

- ❌ Push `.env`, secret, key, dump DB, file build lên repo.
- ❌ `git push origin development` / `git push origin main` trực tiếp.
- ❌ `git push --force` lên branch dùng chung.
- ❌ Commit trực tiếp trên `development` / `main`.
- ❌ Merge branch feature/fix thẳng vào `main`, bỏ qua `development`.
- ❌ Tạo PR release vào `main` khi `development` chưa ổn định hoặc chưa qua review.
- ❌ Dùng `git add .` mà không kiểm tra `git status`.
- ❌ Để branch tồn tại quá lâu không đồng bộ với `development`.
