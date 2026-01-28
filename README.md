
# CHO 세포의 UPR 균형지수와 생산성 분석

## 프로젝트 개요

CHO(Chinese Hamster Ovary) 세포의 **UPR(Unfolded Protein Response) 균형지수**와 **단백질 생산성(Qp)** 간의 상관관계를 분석하고, 머신러닝을 활용하여 고생산/저생산 세포를 분류하는 연구 프로젝트입니다.

## 연구 배경

CHO 세포는 항체와 같은 단백질 의약품 생산에 가장 널리 사용되는 세포주입니다. 그러나 동일한 배양 조건에서도 세포마다 단위 세포당 생산성(Qp)이 크게 달라지며, 이를 예측하기 어렵습니다.

단백질 생산 과정에서 소포체(ER)에 접히지 않은 단백질이 축적되면 ER 스트레스가 유발되고, 이를 완화하기 위해 UPR이 활성화됩니다. UPR에는:
- **생존 경로(Pro-survival)**: 스트레스를 완화하여 세포를 보호
- **세포사멸 경로(Pro-apoptotic)**: 심한 스트레스 시 세포사멸 유도

이 두 경로의 균형이 세포의 운명과 생산성을 결정합니다.

## 연구 목표

1. UPR 균형지수와 secretory 점수를 정의하고, CHO 세포의 Qp와의 관계를 정량적으로 분석
2. 유전자 발현 데이터를 이용한 고생산/저생산 세포 분류 모델 구축
3. Autoencoder 저차원 표현의 예측 유용성 평가

## 데이터셋

- **출처**: GEO 데이터베이스 GSE30321 (Gene expression profiling of CHO production cell lines)
- **플랫폼**: GPL13791 (Affymetrix CHO 전용 마이크로어레이)
- **샘플**: 총 295개 중 169개가 Qp 값 보유
- **분류**: 상위 30%와 하위 30%를 각각 고생산/저생산 그룹으로 분류 (총 103개 샘플 사용)

## 방법론

### 1. 데이터 전처리
- Log1p 변환 및 Z-score 표준화
- 상위 5,000개 가변 유전자 선택

### 2. UPR & Secretory Gene Set 기반 Feature

다음 유전자 세트를 정의하여 pathway feature 생성:

**Pro-survival UPR 유전자**
- HSPA5, HSP90B1, PDIA4, PDIA3, PDIA6, XBP1, ATF6

**Pro-apoptotic UPR 유전자**
- DDIT3, BBC3, BAX, PMAIP1

**Secretory 경로 유전자**
- SEC61A1, SEC24D, SAR1A, EDEM1, HERPUD1, HYOU1

**계산된 지표**
- `uprsurvival`: Pro-survival UPR 점수
- `uprapoptotic`: Pro-apoptotic UPR 점수
- `uprbalance`: uprsurvival - uprapoptotic (균형지수)
- `secretory`: Secretory 경로 점수

### 3. Autoencoder를 이용한 차원 축소
- 구조: 5,000차원 → 256차원 → 2차원 → 256차원 → 5,000차원
- 학습: 50 epoch, Adam optimizer
- 최종 Reconstruction Loss: 0.48

### 4. 분류 모델
- Logistic Regression (pathway features 4개)
- Logistic Regression (Autoencoder latent 2차원)
- MLP (Autoencoder latent 2차원)
- Perceptron (pathway features 4개)

## 주요 결과

### UPR 균형지수와 생산성의 관계
- `uprbalance`와 `secretory` 점수가 Qp와 양의 상관관계
- `uprapoptotic`는 음의 상관관계
- Pro-survival UPR이 활성화되고 pro-apoptotic UPR이 억제될수록 생산성 향상

### 분류 성능 비교

| 모델 | Feature | Accuracy | ROC AUC |
|------|---------|----------|---------|
| Logistic Regression | Pathway (4개) | **1.00** | **1.00** |
| Logistic Regression | Latent (2개) | 0.81 | 0.92 |
| MLP | Latent (2개) | 0.81 | 0.95 |
| Perceptron | Pathway (4개) | 0.71 | - |

**Cross Validation 결과 (Pathway Logistic Regression)**
- 5-fold CV: Accuracy 0.805 ± 0.057, AUC 0.854 ± 0.062
- 10 Random Splits: Accuracy 0.810 ± 0.048, AUC 0.881 ± 0.071

## 파일 구성

```
├── final-code.ipynb          # 전체 분석 코드
├── BMBD_project_report_yunhyeonju.pdf  # 프로젝트 보고서
└── README.md                 # 본 문서
```

## 사용 기술

**Python Libraries**
- `pandas`, `numpy`: 데이터 처리
- `scikit-learn`: 머신러닝 모델 (Logistic Regression, PCA, StandardScaler)
- `torch`: Autoencoder 구현
- `matplotlib`: 시각화

## 실행 방법

### 1. 환경 설정
```bash
pip install numpy pandas matplotlib scikit-learn torch
```

### 2. 데이터 다운로드
- GEO에서 [GSE30321](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE30321) 데이터 다운로드
- `GSE30321_series_matrix.txt`, `GPL13791_datatable.txt` 필요

### 3. 코드 실행
```bash
jupyter notebook final-code.ipynb
```

## 주요 인사이트

1. **단순한 pathway feature만으로도 완벽한 분류 성능** (Accuracy 1.0, AUC 1.0)
2. **UPR 균형지수(uprbalance)와 secretory 점수가 생산성 예측의 핵심**
3. Autoencoder 잠재 표현도 준수한 성능 (AUC 0.92~0.95)
4. 소수의 생물학적으로 의미 있는 feature가 수천 개의 유전자보다 효과적

## 한계점 및 향후 과제

- 제한된 샘플 수 (103개)
- 더 다양한 유전자 세트 및 경로 탐색 필요
- 외부 데이터셋을 통한 검증 필요
- 다른 세포주 및 배양 조건에 대한 일반화 검증

## 참고문헌

1. Clarke, C. et al. (2011). *Predicting cell-specific productivity from CHO gene expression*. Journal of Biotechnology, 151(2), 159-165.
2. Li, Z.-M. et al. (2022). *Factors Affecting the Expression of Recombinant Protein in CHO Cells*. Frontiers in Bioengineering and Biotechnology, 10.
3. Chakrabarti, A., Chen, A. W., Varner, J. D. (2011). *A review of the mammalian unfolded protein response*. Biotechnology and Bioengineering, 108(12), 2777-2793.
4. Prashad, K., Mehra, S. (2015). *Dynamics of unfolded protein response in recombinant CHO cells*. Cytotechnology, 67(2), 237-254.

## 저자

**윤현주** (바이오의공학부, 2023250054)

## 라이선스

본 프로젝트는 학술 연구 목적으로 작성되었습니다.
