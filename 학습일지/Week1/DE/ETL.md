# ETL
raw data를 가져와서, 분석/저장에 쓰기 좋은 형태로 바꾸고, 최종 저장소에 적재하는 전체 흐름
<img width="1466" height="702" alt="image" src="https://github.com/user-attachments/assets/9ef514c2-1d11-44ee-933d-505b17593a20" />

```basic
Extract → Transform → Load
추출 → 변환 → 적재
```

### 장점
> - 데이터 품질 관리 쉬움
> - 분석 데이터 신뢰성 높음
> - 저장소 부담 줄어듦
> - 보안 관리 유리
> - DW에 적합

### **Staging Area ⭐️**

원천 데이터에서 Transform을 위해 중간에 잠깐 데이터를 보관하고 검증하는 공간

> 사용 이유
> 
> 
> : raw 데이터를 수집하는 환경과 데이터를 변환·분석하는 환경을 분리하기 위해
>
