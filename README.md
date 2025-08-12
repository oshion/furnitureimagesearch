# furnitureimagesearch


# Furniture Detection & Search System

## 📌 프로젝트 개요
집 인테리어 사진을 업로드하면 내부 가구를 객체별로 탐지하고,  
유사한 가구 이미지를 검색하는 시스템.

- **이미지 검색**: 업로드한 사진과 비슷한 가구 찾기
- **텍스트 검색**: "흰색 북유럽풍 소파"처럼 텍스트로 가구 찾기
- **실시간 로딩 상태 표시**: 처리 단계를 Front에 표시

---



⚡ 처리 단계 예시
1. 객체 탐지
DETR or Grounding DINO 사용
2. 객체 분리/크롭
필요 시 SAM 사용, 또는 메모리상 crop
3. 임베딩 추출
CLIP / OpenCLIP 사용
4. 유사도 검색
벡터 DB에서 top-k 결과 반환

💡 설계 팁
- 속도 vs 정확도 트레이드오프 고려
  DETR(커스텀 학습): 고정된 클래스 & 높은 정확도
  Grounding DINO: 다양한 클래스 & Zero-shot


📚 참고 모델
- Grounding DINO: 텍스트 기반 객체 탐지
- SAM: 정밀 객체 분할 (선택 사항)
- CLIP / OpenCLIP: 이미지·텍스트 멀티모달 임베딩
- DETR: Transformer 기반 객체 탐지
- YOLO: 실시간 객체 탐지

---



## 🔍 기술 스택 & 모델 비교

### 1. YOLO
- **장점**: 빠름, 구현 쉬움
- **단점**: 클래스 고정, 비슷한 가구 구분 어려움
- **가구 탐지 적합성**: 제한된 클래스 + 실시간 필요 시 사용

### 2. DETR (커스텀 학습)
- **장점**: 높은 정확도, 복잡한 배경/작은 객체 인식 가능
- **단점**: 학습 필요, 실시간 속도는 YOLO보다 느림
- **가구 탐지 적합성**: 고정된 가구 리스트 + 정확도 최우선 시 적합

### 3. Grounding DINO
- **장점**: 텍스트 프롬프트 탐지, Zero-shot 가능, 다양한 가구/스타일 인식
- **단점**: 속도 느림, GPU 메모리 요구 높음
- **가구 탐지 적합성**: 유연한 탐지, 새로운 가구 카테고리 대응 필요 시 최적

---

## 📊 결론 — 혼합 전략
- **이미지 검색**: 커스텀 학습된 DETR 사용 → 정확도 & 속도 최적화
- **텍스트 검색**: Grounding DINO 사용 → Zero-shot & 스타일 검색 가능




---

## 📐 아키텍처 다이어그램

```mermaid
flowchart LR
    subgraph Frontend
        UI[사용자 UI] --> WS[WebSocket 연결]
    end

    subgraph SpringBoot[Spring Boot 서버]
        WS --> CTRL[검색 컨트롤러]
        CTRL --> |이미지 검색| LambdaImage[Lambda: 커스텀 DETR + CLIP]
        CTRL --> |텍스트 검색| LambdaText[Lambda: Grounding DINO + CLIP]
        LambdaImage --> VDB[(벡터 DB)]
        LambdaText --> VDB
        VDB --> CTRL
    end

    subgraph AWS
        LambdaImage
        LambdaText
        S3[(S3: 이미지 저장)]
    end

    CTRL --> UI
