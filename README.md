# bayesianoptimal (BayesianOptimization의 번역)


# Bayesian Optimization

베이지안 추론과 가우시안 프로세스를 기반으로 한 베이지안 글로벌 최적화의 순수한 파이썬 구현입니다.

PyPI (pip)를 통한 설치:

```console
$ pip install bayesian-optimization
```

conda-forge 채널을 통한 Conda 설치:

```console
$ conda install -c conda-forge bayesian-optimization
```

베이지안 추론과 가우시안 프로세스를 기반으로 한 제한 조건이 있는 글로벌 최적화 패키지로, 알려지지 않은 함수의 최대값을 가능한 한 적은 반복으로 찾으려고 합니다. 이 기술은 높은 비용 함수를 최적화하는 데 특히 적합하며, 탐사와 활용 간의 균형이 중요한 상황에서 유용합니다.

## 빠른 시작
아래에 베이지안 최적화 패키지의 기본 사항에 대한 빠른 투어가 있습니다. 더 자세한 정보, 고급 기능 및 사용/구현 팁은 examples 폴더에 있습니다. 제안하는 단계는 다음과 같습니다:

- 가장 중요한 기능을 사용하는 방법을 배우려면 기본 노트북(notebook_기본)을 따라가세요.

- 패키지를 더 유연하게 사용하는 방법, 범주형 매개변수를 처리하는 방법, 관찰자를 사용하는 방법 등을 배우려면 고급 노트북((notebook_고급)을 확인하세요.

- 작동 방식을 단계별로 시각화한 노트북(notebook_시각화)을 살펴보세요.

- 추가 제약 조건이 있을 때, 베이지안 최적화를 사용하는 방법을 이해하기 위해 베이지안 제약 최적화 노트북(notebook_제약)을 확인하세요.

- 탐색과 활용 사이의 균형을 유지하고 제어하는 노트북(notebook_탐색vs활용)을 탐험하세요.

- 교차 검증 및 베이지안 최적화를 사용하여 머신 러닝 모델의 매개변수를 조정하는 방법에 대한 예시를 보려면 파이썬 파일(sklearn_exam)을 참조하세요.

- 매개변수의 경계를 동적으로 변경하여 검색을 가속화하는 방법에 대해 자세히 알아보려면 도메인 노트북(notebook_도메인 가속화)을 탐색하세요.

- 마지막으로, 이 패키지를 사용하여 분산 방식으로 베이지안 최적화를 구현하는 방법에 대한 아이디어를 보려면 파이썬 파일(async_최적화)을 확인하세요.


## 작동 방식
베이지안 최적화는 최적화하려는 함수를 가장 잘 설명하는 함수의 사후 분포(가우시안 프로세스)를 구성함으로써 작동합니다. 관측 횟수가 증가함에 따라 사후 분포가 개선되고 알고리즘은 파라미터 공간에서 어떤 영역을 탐색할 가치가 있는지, 어떤 영역을 탐색하지 않아야 하는지에 대해 더 확신을 가집니다.

사후 분포는 함수의 불확실성을 나타내며, 최적화 알고리즘은 불확실성이 높은 지역을 더 많이 탐사하려고 노력합니다. 이를 통해 빠르게 전역 최대값에 수렴할 수 있습니다. 기본적으로, 알고리즘은 탐험(불확실한 지역 탐색)과 활용(현재로서는 최고로 보이는 지역 탐색) 사이의 균형을 유지하면서 최적화 과정을 진행합니다.

![image](https://github.com/qor6/bayesianoptimal/assets/87318054/271fac58-1e07-43c4-85fc-6dc694ed260a)

알고리즘이 반복됨에 따라, 알고리즘은 대상 함수에 대한 알고 있는 정보를 고려하여 탐사와 활용의 필요를 균형있게 조절합니다. 각 단계에서는 알고리즘이 이전에 탐색한 지점들에 가우시안 프로세스를 적합시키고, 사후 분포는 탐사 전략(예: UCB(상한 신뢰 경계), EI(예상 개선))과 결합되어 다음으로 탐사해야 할 지점을 결정합니다(아래 gif 참조).

각 단계에서 가우시안 프로세스는 관측된 데이터를 기반으로 함수의 현재 추정을 제공합니다. 탐사와 활용의 균형을 맞추기 위해, 탐사적으로 불확실한 영역을 선택하고, 동시에 현재까지 최선으로 간주되는 영역을 활용합니다. 이 방식으로 최적화 알고리즘이 효율적으로 함수의 최대값을 찾아 나갑니다.

![image](https://github.com/qor6/bayesianoptimal/assets/87318054/bdbbf3ae-46a0-4a9a-93c4-a9947cd1f60b)
이 프로세스는 최적의 조합과 가까운 매개변수 조합을 찾기 위해 필요한 단계 수를 최소화하기 위해 설계되었습니다. 이를 위해 이 방법은 최적의 매개변수를 찾는 것이 여전히 어려운 문제임에도 불구하고 더 저렴한(계산상으로) 대리 최적화 문제(획득 함수의 최대값 찾기)를 사용합니다. 따라서 베이지안 최적화는 최적화해야 할 함수를 샘플링하는 것이 매우 비용이 많이 드는 상황에서 가장 적합합니다. 이 방법에 대한 적절한 논의는 참고 자료를 확인하세요.

베이지안 최적화 패키지 기본
===============================================
## 1. 최적화할 함수 정하기
함수 최적화를 위한 것이므로 첫 번째이자 가장 중요한 구성 요소는 당연히 최적화할 함수입니다.

주의: 아래 함수의 출력이 매개변수에 어떻게 의존하는지 정확히 알고 있습니다. 당연히 이것은 예시일 뿐이며 실제 시나리오에서는 알 수 없습니다. 그러나 이 패키지를 사용하기 위해서 (일반적으로 이 기술을 사용하기 위해서) 함수 `f`가 필요한 이유는 알려진 매개변수를 입력으로 받아 실수를 출력하는 함수 `f`뿐이기 때문입니다.

```python
def black_box_function(x, y):
    """최대화하고자 하는 내부가 알려지지 않은 함수.
       이는 단순히 예시로 사용되며 출력 값을 생성하는 프로세스에 대한 내용은
       알려지지 않았다고 생각하면 됩니다.
    """
    return -x ** 2 - (y - 1) ** 2 + 1
```

## 2. 시작하기
시작하려면 최적화할 함수`f`와 해당 매개변수의 경계 `pbounds`를 지정하여 `BayesianOptimization` 객체를 인스턴스화하면 됩니다. 이는 제약 조건이 있는 최적화 기술이므로 작동하려면 각 매개변수에 대해 조사할 수 있는 최소값과 최대값을 지정해야 합니다.
```python
from bayes_opt import BayesianOptimization

# 매개변수 공간의 경계 영역
pbounds = {'x': (2, 4), 'y': (-3, 3)}

optimizer = BayesianOptimization(
    f=black_box_function,
    pbounds=pbounds,
    random_state=1,
)
```
BayesianOptimization 객체는 조정이 거의 필요 없이 즉시 작동합니다. 알아두어야 할 주요 메서드는 명백한 대로 수행하는 `maximize` 입니다.

maximize에 전달할 수 있는 매개변수가 많지만, 그 중에서도 가장 중요한 것은:
- `n_iter`: Bayesian 최적화 단계를 몇 번 수행할지를 나타냅니다. 단계가 많을수록 좋은 최대값을 찾을 확률이 높아집니다.
- `nit_points`: 무작위 탐사의 단계를 몇 번 수행할지를 나타냅니다. 무작위 탐사는 탐사 공간을 다양화시키는 데 도움이 될 수 있습니다.

```python
optimizer.maximize(
    init_points=2,
    n_iter=3,
)
```

    |   iter    |  target   |     x     |     y     |
    -------------------------------------------------
    |  1        | -7.135    |  2.834    |  1.322    |
    |  2        | -7.78     |  2.0      | -1.186    |
    |  3        | -19.0     |  4.0      |  3.0      |
    |  4        | -16.3     |  2.378    | -2.413    |
    |  5        | -4.441    |  2.105    | -0.005822 |
    =================================================



발견된 parameters와 target 값의 최상의 조합은 `optimizer.max` 속성을 통해 액세스할 수 있습니다.
```python
print(optimizer.max)
>>> {'target': -4.441293113411222, 'params': {'y': -0.005822117636089974, 'x': 2.104665051994087}}
```

조사된 모든 parameters의 목록과 해당 target 값은 `optimizer.res` 속성을 통해 사용 할 수 있습니다.
```python
for i, res in enumerate(optimizer.res):
    print("Iteration {}: \n\t{}".format(i, res))

>>> Iteration 0:
>>>     {'target': -7.135455292718879, 'params': {'y': 1.3219469606529488, 'x': 2.8340440094051482}}
>>> Iteration 1:
>>>     {'target': -7.779531005607566, 'params': {'y': -1.1860045642089614, 'x': 2.0002287496346898}}
>>> Iteration 2:
>>>     {'target': -19.0, 'params': {'y': 3.0, 'x': 4.0}}
>>> Iteration 3:
>>>     {'target': -16.29839645063864, 'params': {'y': -2.412527795983739, 'x': 2.3776144540856503}}
>>> Iteration 4:
>>>     {'target': -4.441293113411222, 'params': {'y': -0.005822117636089974, 'x': 2.104665051994087}}
```



