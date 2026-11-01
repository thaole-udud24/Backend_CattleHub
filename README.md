# cattle-farm-backend

Node.js + Express + MongoDB (Mongoose), CommonJS, zod.

```bash
npm i
cp .env.example .env      # điền JWT_ACCESS_SECRET, OTP_SECRET, MONGODB_URI
npm run seed              # tạo admin đầu tiên (ADMIN_EMAIL / ADMIN_PASSWORD)
npm run dev               # GET /api/v1/health
npm run lint              # kiểm tra ESLint toàn dự án
```

## Quy trình làm việc

- Luồng code: branch riêng → PR vào `development` → PR release vào `main`. Không push thẳng `development`/`main`.
- Commit theo Conventional Commits, mô tả rõ ràng, ví dụ `fix(order): return 409 instead of 500 when order code already exists`.
- Khi `npm i`, husky tự cài hook pre-commit: chạy `eslint --fix` trên file `.js` đã stage, lỗi lint sẽ chặn commit.
- Không commit `.env` hay secret; chỉ cập nhật `.env.example`.

Chi tiết xem [GIT_WORKFLOW.md](GIT_WORKFLOW.md).

Lưu ý: giao dịch (tạo đơn + snapshot phí + giữ tồn kho) cần MongoDB replica set.

Sau này cần thêm khi làm các module khác: `npm i multer exceljs pdfkit node-cron decimal.js google-auth-library`
