# 다변량 통계학

# 모집단 평균벡터에 관한 추론

# -SAS

1. **일변량 평균 검정**
- Sweat 데이터에 대한 일변량 평균 검정 수행
- 데이터 불러오기

- SAS
proc import datafile = "C:\Users\samsung\Desktop\sweat.csv" dbms=csv
replace out = sweat; 
getnames = yes; 
run; 

proc print data = sweat; 
run;

- means procedure를 이용하여 데이터의 평균벡터 확인

```sass
proc means data= sweat; 
    var sweatrate; 
run;
```

- 일변량 평균 검정의 경우 univariate 혹은 ttest procedure를 이용할 수 있다.

```sass
proc univariate data = sweat mu0=4; 
    var sweatrate; 
run;
 
proc ttest data = sweat h0 = 4; 
    var sweatrate; 
run;
```

1. **다변량 평균 검정**
- 위의 sweat 데이터에서 sweatrate, sodium, potassium 열에 대한 평균 검정 수행

```sass
proc means data= sweat; 
    var sweatrate sodium potassium; 
run;
```

- 가설 $H_0:\mu=(4,50,10), H_1: \pi\ne(4,50,10)$에 대해 검정하는 것이 목적
- Case 1: 직접 계산
    - 먼저 covariance matrix를 계산
    
    ```sass
    proc corr data= sweat cov outp = sweat_corr; 
     var sweatrate sodium potassium; 
    run;
    ```
    
    - Covariance matrix를 iml procedure에서 사용할 수 있도록 변형
    
    ```sass
    data sweat_cov_mat; 
     set sweat_corr;
     if _type_ = 'COV'; 
     drop _character_; 
    run
    ```
    
    - T 계산
    
    ```sass
    proc iml; 
     mu = {0.64, -4.6, -0.035}; 
     use sweat_cov_mat; 
     ead all into cov_mat; 
     mu_t = mu`;
     inv_cov = inv(cov_mat); 
     T = 20 * mu_t * inv_cov * mu; 
     q1 = 19 * 3 / (20 - 3) * finv(.95, 3, 17); 
     q2 = 19 * 3 / (20 - 3) * finv(.90, 3, 17);
     print cov_mat, inv_cov, mu, T, q1, q2; 
    run;
    ```
    
- Case 2: proc glm 이용
    - 먼저 귀무가설을 이용하여 데이터를 변형
    
    ```sass
    data sweat_h; 
     set sweat;
     nx1 = sweatrate -4; 
     nx2 = sodium - 50;
     nx3 = potassium - 10; 
    run;
    
    proc print data = sweat_h; 
    run;
    ```
    
    - proc glm을 통해 Wilks lambda값을 계산
    
    ```sass
    proc glm data = sweat_h;
     model nx1 nx2 nx3 = / ; 
     manova h = intercept; 
    run;
    ```
    
    - Wilks lambda 값과 iml procedure를 이용하여 $T^2$를 계산
    
    ```sass
    proc iml; 
     T = 19 * (1-0.66112774)/0.66112774; 
     print T; 
    run;
    ```
    
    - 3차원 산점도 출력
    
    ```sass
    proc G3D data=sweat;
        scatter sodium * sweatrate = potassium;
    run;
    ```
    
1. **두 모집단에 대한 검정**
- 두 모집단에 대한 검정에서도 직접  $T^2$ 계산
- 각 집단의 공분산, 평균은 다음처럼 사용할 수 있다.
- 데이터 불러오기
