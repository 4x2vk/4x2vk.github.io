---
layout: post
title: "[Project] 클래스를 적용해 기본적인 연산을 수행할 수 있는 계산기 만들기"
date: 2025-07-15 20:18 +0900
categories: [BootCamp, TIL]
tags: [project, java]
---

부트캠프에서 다양한 프로젝트를 진행하고 있는데 그중 가장 기본적인 과제를 통해 내가 언어를 제대로 익혔는지 확인할 수 있었습니다. 그 프로젝트는 바로 계산기였습니다.

예전에는 다른 언어로 계산기를 만들 때 그렇게 중요하게 생각하지 않았었습니다. 하지만 이번에는 코드의 모든 세부사항이 정말 중요하다는 걸 깨달았습니다. 현재 3단계에서 2단계까지 도달했고 최대한 코드를 더 깔끔하고 읽기 쉽게 만들려고 노력하고 있습니다. 왜냐하면 내 경험상 새로운 사람이 내 코드를 보게 되었을 때 그 사람이 내 코드를 쉽게 이해하고 수정할 수 있어야 하기 때문이라고 봅니다. 

코드는 아직도 수정중이지만...

#### Calculator class

```
package level2;

import java.util.ArrayList;
import java.util.List;

public class Calculator {
    private List<Double> results = new ArrayList<>();

    private double num1, num2;
    private char operator;

    public void setOperator(char operator) {
        this.operator = operator;
    }

    public char getOperator() {
        return this.operator;
    }

    public void setNum1(double value) {
        this.num1 = value;
    }

    public void setNum2(double value) {
        this.num2 = value;
    }

    public double getNum1() {
        return this.num1;
    }

    public double getNum2() {
        return this.num2;
    }

    public List<Double> getResults() {
        return this.results;
    }

    public void removeFirstResult() {
        if (!results.isEmpty()) {
            results.remove(0);
        }
    }

    public double calculate() {
        double result;
        switch (operator) {
            case '+':
                result = num1 + num2;
                break;
            case '-':
                result = num1 - num2;
                break;
            case '*':
                result = num1 * num2;
                break;
            case '/':
            case ':':
                if (num2 == 0) throw new ArithmeticException("\uD83D\uDEAB 0으로 나눌 수 없습니다!");
                result = num1 / num2;
                break;
            default:
                throw new IllegalArgumentException("\uD83D\uDEAB 올바르지 않은 연산자입니다. 다시 입력해주세요!");
        }
        results.add(result);
        return result;
    }
}
```

이번에 가장 고민했던 부분은 throw new ArithmeticException을 배운 것이었어요. 숫자를 0으로 나눌 때

![](https://velog.velcdn.com/images/alphasapiens/post/81daa14f-27dc-4d2c-bca8-4a2c9dc2ce6d/image.png)


이 로직을 통해 `🗑️ 0으로 나눌 수 없습니다!`라는 메시지가 나올 줄 알았는데 위에 보이는 결과처럼 나오더라고요. 그래서 이 부분에 대해 고민을 많이 했던 것 같습니다.


처음에는 결과가 나오는 부분을 이렇게 구현했는데...

![](https://velog.velcdn.com/images/alphasapiens/post/0d46ea49-06d4-4530-8cde-3632d445cad3/image.png)

나중에 알고 보니, 이것도 `try-catch`로 한 번 더 예외를 잡아줘야 하는 거였네요

like this way ↓

![](https://velog.velcdn.com/images/alphasapiens/post/428dec37-846d-45a4-bd36-06d527a5c5c1/image.png)


App main

```
package level2;

import java.util.Scanner;

public class App {
    public static void main(String[] args) {
        Calculator calculator = new Calculator();

        Scanner input = new Scanner(System.in);

        while(true){
            System.out.println("                              ");
            System.out.println("==============================");
            System.out.println("✨     SIMPLE CALCULATOR    ✨");
            System.out.println("==============================");
            System.out.println("       'exit' 입력하면 종료     ");
            System.out.println("      'delete' 입력하면 종료     ");
            System.out.println("                              ");

            System.out.print("👉 1번째 숫자 입력: ");
            String firstInput = input.nextLine();
            if(firstInput.equals("exit")) break;
            if (firstInput.equals("delete")) {
                calculator.removeFirstResult();
                System.out.println("🗑️ 첫 결과가 삭제되었습니다!");
                continue;
            }

            System.out.print("👉 2번째 숫자 입력: ");
            String secondInput = input.nextLine();
            if(secondInput.equals("exit")) break;
            if (secondInput.equals("delete")) {
                calculator.removeFirstResult();
                System.out.println("🗑️ 첫 결과가 삭제되었습니다!");
                continue;
            }

            double num1, num2;

            try {
                num1 = Double.parseDouble(firstInput);
                num2 = Double.parseDouble(secondInput);
                if (num1 < 0 || num2 < 0) {
                    System.out.println("\uD83D\uDEAB 양수만 입력해주세요!");
                    continue;
                }
            } catch (NumberFormatException e) {
                System.out.println("\uD83D\uDEAB 숫자만 입력해주세요!");
                continue;
            }

            System.out.print("사용 가능한 연산자 [+  -  *  / ]: ");
            String operator = input.nextLine();
            if(operator.equals("exit")) break;
            if (operator.equals("delete")) {
                calculator.removeFirstResult();
                System.out.println("🗑️ 첫 결과가 삭제되었습니다!");
                continue;
            }
            char op = operator.charAt(0);

            calculator.setNum1(num1);
            calculator.setNum2(num2);
            calculator.setOperator(op);
            
            try {
                double result = calculator.calculate();
                System.out.println("==============================");
                System.out.println("🧾 결과: " + calculator.getNum1() + " " + calculator.getOperator() + " " + calculator.getNum2() + " = " + result);
                System.out.println("==============================");
            } catch (ArithmeticException | IllegalArgumentException e) {
                System.out.println(e.getMessage());
            }
        }
        System.out.println("👋 계산기를 종료합니다. 감사합니다!");
        input.close();
    }
}
```
---
![](https://velog.velcdn.com/images/alphasapiens/post/ff0301a5-721f-449a-be77-d41dfc0a9462/image.png)

특히 IntelliJ에서 뜨는 경고 메시지를 확인하면서, 코드의 잘못된 부분을 하나씩 고쳐 나갈 수 있었던 점이 정말 도움이 되었다고 느꼈습니다.

아직 많이 부족하지만, 하나하나 고쳐가면서 점점 더 나아지고 있다는 느낌을 받고 있습니다.