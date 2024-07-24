# 다변량 통계학

# 모집단 평균벡터에 관한 추론

# -SAS

1. **일변량 평균 검정**
- Sweat 데이터에 대한 일변량 평균 검정 수행
- 데이터 불러오기
```sass
proc import datafile = "C:\Users\samsung\Desktop\sweat.csv" dbms
=csv
    replace out = sweat; 
    getnames = yes; 
run; 

proc print data = sweat; 
run;
```

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

2. **다변량 평균 검정**
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
    
3. **두 모집단에 대한 검정**
- 두 모집단에 대한 검정에서도 직접  $T^2$ 계산
- 각 집단의 공분산, 평균은 다음처럼 사용할 수 있다.
- 데이터 불러오기
```sass
proc import datafile = 'C:\Users\samsung\Desktop\man.csv'
out = man replace; 
proc import datafile = 'C:\Users\samsung\Desktop\woman.csv'
out = woman replace; 
run;
```

- corr procedure로 covariance를 계산, 이를 하나의 데이터로 저장
```sass
proc corr data = man cov outp = out_man;
proc corr data = woman cov outp = out_woman; 
run; 
data out_man_woman ; 
   set out_man out_woman; 
run;
```

- iml procedure를 이용, $T^2$와 $F$값을 계산할 수 있다.
```sass
proc iml ; 
    use out_man_woman ; 
    p = 1:4; 
    q = 12:15; 
    r = 5;
    s = 16; 
    read point p into s_man; 
    read point q into s_woman; 
    read point r into x_man ; 
    read point s into x_woman ;
    x_man_t = x_man`;
    x_woman_t = x_woman`;
    Sp = ((32-1) # s_man _ (32-1) # s_woman) / (32 + 32 -2);
    Inv_Sp = inv(Sp);
    T2 = 1(1/32 + 1/32) * (x_man – x_woman) * inv_Sp * (x_man_t –
    x_woman_t);
    F = ((32 + 32 – 2) * 4) / (32 + 32 -4 – 1) * finv(.95, 4, 
    59);
    Print T2, F;
run;
```

# -R

- 필요패키지 불러오기
```r
library(scatterplot3d)
library(dplyr)
```

1. **일변량 평균 검정**
- 데이터 불러오기
```r
## outpath는 미리 지정한 저장경로
sweat <- read.csv(paste0(outpath, "sweat.csv")) %>%
select(-obs)
```

- mean()함수를 통한 데이터의 평균과 표준편차 확인
```r
sweatrate <- sweat[, 1]
data.frame(mean = mean(sweatrate), sd = sd(sweatrate))
```

- 일변량 평균 검정을 하기 위한 t.test() 이용
```r
t.test(sweatrate, mu = 4)
```
2. **다변량 평균 검정**
- sweat 데이터에 대한 평균과 표준편차 계산
```r
## apply() 함수를 통해 행렬 데이터의 열평균과 열표준편차를 계산 가능
sweat_stat <- apply(sweat, 2, function(x) {
c(mean(x), sd(x))
})
rownames(sweat_stat) <- c("mean", "sd")
sweat_stat
```

- 가설 $H_0:\mu=(4,50,10), H_1: \pi\ne(4,50,10)$에 대한 검정 실시
```r
mu0 <- c(4, 50, 10)
mu_test <- sweat_stat[1, ] - mu0
cov_sweat <- cov(sweat)

## 관측치 개수
n <- dim(sweat)[[1]]

## 검정하려는 벡터의 길이
p <- length(mu_test)
t_value <- n %*% t(mu_test) %*% solve(cov_sweat) %*% mu_test
f_95 <- (n - 1) * p / (n - p) * qf(.95, p, n - p)
f_90 <- (n - 1) * p / (n - p) * qf(.90, p, n - p)
data.frame(t_value = t_value, f_95 = f_95, f_90 = f_90)
```

3. **두 모집단에 대한 검정**
- 두 모집단에 대한 검정에서도 $T^2$를 계산 할 수 있다.
- 데이터를 불러온 후 공분산을 다음과 같이 저장

```r
man <- read.csv(paste0(outpath, "man.csv"))
woman <- read.csv(paste0(outpath, "woman.csv"))

man_mean <- colMeans(man)
woman_mean <- colMeans(woman)

man_cov <- cov(man)
woman_cov <- cov(woman)

n1 <- dim(man)[[1]]
n2 <- dim(woman)[[1]]

p <- length(man_mean)
```

- 행렬연산을 통해 다음과 같이  $S_p,T^2$를 계산 할 수 있다.
```r
Sp <- ((n1 - 1) * man_cov + (n2 - 1) * woman_cov) / (n1 + n2 - 2)
T2 <- 1 / (1/n1 + 1/n2) * t(man_mean - woman_mean) %*% solve(Sp) %*% (man_mean - woman_mean)
F_val = ((n1 + n2 - 2) * p) / (n1 + n2 - p - 1) * qf(.95, p, n1 + n2 - p -1)

data.frame(T2 = T2, F_val = F_val)
```
