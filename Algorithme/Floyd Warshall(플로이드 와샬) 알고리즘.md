그래프 문제를 해결할 때 사용하기 유용한 알고리즘이다.

[백준 경로찾기 문제](https://www.acmicpc.net/problem/11403)

지점 I 에서 J 까지 가는길이 있는지 구할 수 있고 I 부터 J 까지의 최솟값 또한 구할 수 있다.
다만(n3)의 효율성을 가진다.

I에서 K 까지 갈 수 있고 K 부터 J까지 갈 수 있다면 I 부터 J 까지 갈 수 있다.

우선 거쳐갈 K를 잡아두고 I에서 K를 갈 수 있는지 그리고 K에서 J를 갈 수 있는지 검사한다.
만약 갈 수 있다면 경로를 추가해준다.

```JAVA
    public static void floydWarshall() {
        for(int k = 0; k < n; ++k)
            for(int i = 0; i < n ; ++i)
                for(int j = 0; j < n; ++j)
                    if(result[i][k] == 1 && result[k][j] == 1)
                        result[i][j] = 1;
    }
```

최솟값을 구하고 싶다면 다음과 같이 값을 바꿔주면 된다.

```JAVA
    public static void floydWarshall() {
        for(int k = 0; k < n; ++k)
            for(int i = 0; i < n ; ++i)
                for(int j = 0; j < n; ++j)
                    if(result[i][k] != 0 && result[k][j] != 0 &&
                    result[i][k] + result[k][j] < result[i][j])
                        result[i][j] = result[i][k] + result[k][j];
    }
```
