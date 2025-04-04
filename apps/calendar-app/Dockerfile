# 1. Builder stage: Node.js 환경에서 앱 빌드
FROM node:18-alpine AS builder

WORKDIR /app

# pnpm 설치
RUN npm install -g pnpm

# 루트 package.json, pnpm-lock.yaml, pnpm-workspace.yaml 복사
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml ./

# 필요한 앱과 패키지 소스 코드 **먼저** 복사
# 중요: .dockerignore 파일에 node_modules가 포함되어 있는지 확인하세요!
COPY . .

# 전체 워크스페이스 의존성 설치 (Turborepo 캐싱 활용)
# COPY . . 이후에 실행하도록 순서 변경
RUN pnpm install --frozen-lockfile

# calendar-app 빌드 (Standalone output 활용)
RUN pnpm --filter=calendar-app build

# 2. Runner stage: Nginx 환경에서 앱 실행
FROM nginx:1.25-alpine AS runner
WORKDIR /app

# Nginx 설정 파일 복사
COPY apps/calendar-app/nginx.conf /etc/nginx/nginx.conf

# Builder 스테이지에서 빌드된 결과물 복사
COPY --from=builder /app/apps/calendar-app/.next/standalone ./
COPY --from=builder /app/apps/calendar-app/.next/static ./.next/static
COPY --from=builder /app/apps/calendar-app/public ./public

# Node.js 서버 실행 (Standalone 모드)
EXPOSE 80
CMD ["node", "server.js"]