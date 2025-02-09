## UI 컴포넌트 테스트

### UI 컴포넌트 테스트가 필요한 이유

UI를 구성하는 컴포넌트의 최소 단위는 버튼과 같은 개별 UI이며, 여러 컴포넌트를 조합하여 중간 → 화면 단위의 UI를 완성한다. 만약 중간 단계의 UI에서 문제가 생기면 최악의 경우 페이지 전체에 문제가 생겨 애플리케이션 사용에 문제를 일으키게 된다.

### 웹 접근성 테스트

웹 접근성이란 신체적, 정신적 특성의 차이나 장애 여부에 관계없이 모든 사용자가 동등하게 웹 서비스와 정보에 접근하고 이용할 수 있는 특성이다. UI 컴포넌트 테스트를 활용하면, 테스트 시에 웹 접근성에 기반한 쿼리로 테스트를 작성해야 하기 때문에 자연스레 웹 접근성 품질도 향상된다.

## UI 테스트 환경 구축

### 테스팅 라이브러리의 역할

- UI 컴포넌트 렌더링
- 렌더링된 요소에서 자식 요소 취득
- 렌더링된 요소에 인터랙션 일으킴

### 필요 패키지 설치

```bash
pnpm add -D jest-environment-jsdom @testing-library/react @testing-library/jest-dom @testing-library/uesr-event

```

### jest.config.ts

```jsx
import type { Config } from "jest";

const config: Config = {
  preset: "ts-jest", // TypeScript 환경을 위한 preset
  testEnvironment: "jest-environment-jsdom", // 가상 DOM 환경
  moduleFileExtensions: ["ts", "js"], // 모듈 파일 확장자
  testMatch: ["**/tests/**/*.test.ts"], // 테스트 파일 매칭 패턴
  transform: {
    "^.+\\\\.ts$": "ts-jest", // .ts 파일을 ts-jest로 변환
  },
  moduleNameMapper: {
    "^@src/(.*)$": "<rootDir>/src/$1", // @src 경로 매핑 추가
  },
  moduleDirectories: ["node_modules", "src"], // 모듈 해석 경로
  collectCoverage: true, // 코드 커버리지 수집
  coverageDirectory: "coverage", // 커버리지 리포트 생성 위치
  coverageReporters: ["json", "lcov", "text", "clover"], // 커버리지 리포트 형식
  setupFilesAfterEnv: ["<rootDir>/setupTests.ts"], // 테스트 환경 설정 파일
  verbose: true, // 상세한 테스트 결과 출력
};

export default config;
```

⚠️만약 `Next.js` 프레임워크처럼 서버와 클라이언트 코드가 공존할 경우 테스트 파일 첫 줄에 주석으로 테스트 환경을 구성할 수 있다.

```jsx
/**
* @jest-environment jet-environment-jsdom
* /
```

### setupTest, tsconfig

`jest.config.ts` 파일에서 설정한 전처리 파일을 생성한다. 매번 테스트할 시 jest-dom 패키지를 import하는 것을 방지할 수 있다.

```tsx
//📁setupTests.ts
import "@testing-library/jest-dom";
```

```tsx
{
  "compilerOptions": {
    "types": ["jest", "@testing-library/jest-dom"],
		//..기존 내용..
  }
}

```

## UI 실전 테스트

기본적인 UI 컴포넌트를 테스트한다.

### 렌더링

`render` 함수를 사용해 jsdom에 테스트할 컴포넌트를 렌더링한다.

```tsx
import { render, screen } from "@testing-library/react";
import { Form } from "@components/Form";

describe("Form 테스트", () => {
  it("이름을 표시한다.", () => {
    render(<Form name="Mario" />);
  });
});
```

### DOM 요소 취득하기

렌더링된 요소 중 특정 DOM 요소를 취득하려면 `screen.getByText`를 사용한다.

실제 DOM에 특정 텍스트가 포함되었는지 확인하는 방법은 `toBeInTheDocument` 함수를 사용한다.

```tsx
import "@testing-library/jest-dom";
import { render, screen } from "@testing-library/react";
import { Form } from "@components/Form";

describe("Form 테스트", () => {
  it("이름을 표시한다.", () => {
    render(<Form name="Mario" />);
    expect(screen.getByText("Mario")).toBeInTheDocument();
  });
});
```

### getByRole

각 HTML 태그 마다 암묵적인 role이 부여되어 있는데 이를 활용해 특정 DOM 요소의 내용을 테스트할 수 있다.

```tsx
it("heading을 표시한다.", () => {
  render(<Form name="Mario" />);
  //heading level 3 -> <h3>..</h3>
  expect(screen.getByRole("heading", { level: 3 }));
});
```

### 이벤트 핸틀러 호출 테스트

아래 실제 테스트 대상 코드를 보면 이벤트 핸들러가 존재하는데 실제 동작에서는 버튼을 클릭해야 `handleSubmit` 핸들러가 동작한다. 테스트 환경에서는 직접 버튼을 클릭할 수 없기 때문에 fireEvent에서 제공하는 목 함수를 이용한다.

```tsx
import React from "react";

export const Form = ({ name }: { name: string }) => {
  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    console.log("Form submitted");
  };

  return (
    <form onSubmit={handleSubmit}>
      <h2>Form</h2>
      <h3>{name}</h3>
      <input type="text" />
      <button type="submit">Submit</button>
    </form>
  );
};
```

```tsx
it('"Submit" 버튼을 클릭하면 이벤트 핸들러가 실행된다..', () => {
  const mockFn = jest.fn();
  render(<Form name="Mario" callback={mockFn} />);
  fireEvent.click(screen.getByRole("button", { name: "Submit" }));
  expect(mockFn).toHaveBeenCalled();
});
```

### 아이템 요소 테스트

다수의 아이템을 갖는 태그(dl, ul, ol)의 자식 요소(li)는 암묵적 역할로 `listitem`을 갖는다.

```tsx
/** 테스트 대상 코드 **/
import React from "react";

type Article = {
  id: number;
  title: string;
  content: string;
};

interface ArticleProps {
  articles?: Article[];
}

export const Article = ({ articles }: ArticleProps) => {
  if (!articles || articles.length === 0) {
    return <div>게시글이 없습니다.</div>;
  }
  return (
    <div>
      <h2>MARIO NEWS!!</h2>
      <ol>
        {articles?.map((article) => (
          <li key={article.id}>
            <p style={{ listStyle: "none", fontWeight: "bold" }}>
              {article.title}
            </p>
            <p>{article.content}</p>
          </li>
        ))}
      </ol>
    </div>
  );
};
```

```tsx
import { render, screen } from "@testing-library/react";
import { Article } from "@components/Article";

const MOCK_ARTICLES = [
  {
    id: 1,
    title: "슈퍼 마리오 오디세이 2 출시 임박",
    content: "닌텐도가 슈퍼 마리오의 새로운 모험을 예고했습니다.",
  },
];

describe("Article 테스트", () => {
  it("목록을 표시한다.", () => {
    render(<Article articles={MOCK_ARTICLES} />);
    expect(screen.getByRole("list")).toBeInTheDocument();
  });

  it("기사 갯수 만큼 목록을 표시한다..", () => {
    render(<Article articles={MOCK_ARTICLES} />);
    expect(screen.getAllByRole("listitem")).toHaveLength(MOCK_ARTICLES.length);
  });
});
```

**부재 목록 테스트**

만약 목록이 없는 경우를 테스트할 때 기존 getByRole, getByLabelText로 테스트를 한다면 테스트 환경에서 취득할 수 없기 때문에 오류가 발생한다. `queryByRole` 함수로 대체하면 된다.

- **queryBy~:** 발견된 요소가 없으면 null을 반환
- **getBy~:** 발견된 요소가 없으면 에러 throw

```tsx
test("목록에 표시할 데이터가 없으면 '게시글이 없습니다.'를 표시한다.", () => {
  render(<Article articles={[]} />);
  const $list = screen.queryByRole("list");
  expect($list).not.toBeInTheDocument();
  expect(screen.getByText("게시글이 없습니다.")).toBeInTheDocument();
});
```

### 테스트 쿼리 우선 순위

테스팅 라이브러리의 요소 취득 API는 다음과 같은 순서로 사용할 것을 권장한다.

1. **모두가 접근 가능한 쿼리**
   - getByRole
   - getByLAbelText
   - getByPlaceholderText
   - getByText
   - getByDisplayValue
2. **시맨틱 쿼리**
   - getByAltText
   - getByTitle
3. **테스트 ID (테스트 작성자가 직접 부여한 요소)**
   - getByTestId

## 인터랙티브 UI 테스트

접근성 기반 쿼리를 이용해 UI 컴포넌트 테스트를 수행한다.

### 접근성 쿼리

```tsx
import React from "react";

interface ContractFormProps {
  handleChange?: (e: React.ChangeEvent<HTMLInputElement>) => void;
}

export const ContractForm = ({ handleChange }: ContractFormProps) => {
  return (
    <fieldset>
      <legend>이용약관 동의</legend>
      <div role="group" aria-labelledby="contract-title">
        <label>
          <input
            type="checkbox"
            onChange={(e) => handleChange?.(e)}
            aria-label="이용약관에 동의합니다"
            role="checkbox"
          />
          이용약관에 동의합니다
        </label>
      </div>
    </fieldset>
  );
};
```

위와 같이 간단한 이용약관 폼을 구성하는 컴포넌트이다. 일반적으로 요소를 묶을 때 `div` 태그를 많이 사용하지만 테스트 시 접근성 기반 쿼리를 사용하려면 의미에 맞는 태그를 최대한 활용하는 것이 중요하다.

테스트 코드 작성 시 웹 접근성이 향상되는 이유는 다음과 같이 테스트 수행 코드를 위해 접근성을 고려하기 때문이다.

```tsx
import { render, screen } from "@testing-library/react";
import { ContractForm } from "@components/ContractForm";

describe("ContractForm 테스트", () => {
  it("이용약관 동의 Form을 렌더링한다.", () => {
    render(<ContractForm />);
    //div 태그로는 쿼리로 가져올 수 없음!
    expect(
      screen.getByRole("group", { name: "이용약관 동의" })
    ).toBeInTheDocument();
  });
});
```

### 입력 이벤트 테스트

입력 이벤트를 테스트하는 방법은 `fireEvent` 함수로도 가능하지만, 실제 사용자 작동과 가깝게 재현하려면 `userEvent` API를 사용하는 것을 추천한다.

`userEvent`를 사용한 모든 인터랙션은 입력이 완료될 때 까지 기다리는 비동기 처리이므로 `awiat` 을 사용해 입력이 완료될 때까지 기다린다.

```tsx
import userEvent from "@testing-library/user-event";
import { render, screen } from "@testing-library/react";
import { SignForm } from "@components/SignForm";

//userEvent 설정
const user = userEvent.setup();

test("메일 주소를 입력하면 화면에 입력된 메일 주소가 표시된다.", async () => {
  const email = "test@example.com";
  render(<SignForm />);
  const $emailInput = screen.getByRole("textbox", { name: "이메일 입력" });
  await user.type($emailInput, email);
  expect($emailInput).toHaveValue(email);
  //초깃값이 입력된 폼 요소가 존재하는지 검증
  expect(screen.getByDisplayValue(email)).toBeInTheDocument();
});
```

**⚠️ `textbox` 로 취급되지 않는 input 테스트**

input 요소 타입에서 password, email처럼 textbox로 취급되지 않는 요소들은 다른 방식으로 요소를 취득해야 한다. 웹 접근성을 지키며 쉽게 취득할 수 있는 방법은 `getByPlaceholderText`가 있다.

```tsx
export const SignForm = ({ handleSubmit, handleChange }: SignFormProps) => {
  return (
    <form onSubmit={(e) => handleSubmit?.(e)}>
					...
          <div>
            <label htmlFor="password">비밀번호</label>
            <input
              id="password"
              type="password"
              onChange={(e) => handleChange?.(e)}
              aria-label="비밀번호 입력"
              aria-required="true"
              placeholder="비밀번호 입력"
              required
            />
          </div>
          ...
        </div>
      </fieldset>
    </form>
  );
};

```

```tsx
test("비밀번호를 입력하면 화면에 입력된 비밀번호가 표시된다.", async () => {
  const password = "password";
  render(<SignForm />);
  const $passwordInput = screen.getByPlaceholderText("비밀번호 입력");
  await user.type($passwordInput, password);
  expect($passwordInput).toHaveValue(password);
  expect(screen.getByDisplayValue(password)).toBeInTheDocument();
});
```

### 클릭 테스트

체크박스 클릭 동작도 `userEvent` API를 활용해 테스트할 수 있다.

```tsx
import { render, screen } from "@testing-library/react";
import { ContractForm } from "@components/ContractForm";
import userEvent from "@testing-library/user-event";

const user = userEvent.setup();

describe("ContractForm 테스트", () => {
  it("이용약관 동의 Form을 렌더링한다.", () => {
    render(<ContractForm />);
    expect(
      screen.getByRole("group", { name: "이용약관 동의" })
    ).toBeInTheDocument();
  });

  it("초기 약관 동의 버튼은 비활성화 상태이다.", () => {
    render(<ContractForm />);
    expect(screen.getByRole("button", { name: "다음" })).toBeDisabled();
  });

  it("약관 동의 체크박스를 클릭하면 버튼이 활성화된다.", async () => {
    render(<ContractForm />);
    const $checkbox = screen.getByRole("checkbox", {
      name: "이용약관에 동의합니다",
    });
    await user.click($checkbox);
    expect(screen.getByRole("button", { name: "다음" })).toBeEnabled();
  });
});
```

## 비동기 처리가 포함된 UI 컴포넌트 테스트

**목 함수 작성**

```tsx
import * as Fetchers from ".";
import { httpError, postMyAddressMock } from "./fixtures";

export function mockPostMyAddress(status = 200) {
  if (status > 299) {
    return jest
      .spyOn(Fetchers, "postMyAddress")
      .mockRejectedValueOnce(httpError);
  }
  return jest
    .spyOn(Fetchers, "postMyAddress")
    .mockResolvedValueOnce(postMyAddressMock);
}
```

**응답 성공 테스트**

```tsx
test("배송 등록이 성공하면 '등록됐습니다'가 표시된다", async () => {
  const mockFn = mockPostMyAddress();
  render(<RegisterAddress />);
  const submitValues = await fillValuesAndSubmit();
  expect(mockFn).toHaveBeenCalledWith(expect.objectContaining(submitValues));
  expect(screen.getByText("등록됐습니다")).toBeInTheDocument();
});
```

**응답 실패 테스트**

```tsx
test("실패하면 '등록에 실패했습니다'가 표시된다", async () => {
  const mockFn = mockPostMyAddress(500);
  render(<RegisterAddress />);
  const submitValues = await fillValuesAndSubmit();
  expect(mockFn).toHaveBeenCalledWith(expect.objectContaining(submitValues));
  expect(screen.getByText("등록에 실패했습니다")).toBeInTheDocument();
});
```

## 스냅샷 테스트

### 스냇샵 기록하기

스냇샵을 기록하면 해당 컴포넌트 파일에 `toMatchSnapshot`을 사용하는 단언문을 실행한다.

```tsx
import { render, screen, fireEvent } from "@testing-library/react";
import { Form } from "@components/Form";

describe("Form 테스트", () => {
  it("Snapshot: Form 컴포넌트가 렌더링 되면 이름이 표시된다.", () => {
    const { container } = render(<Form name="Mario" />);
    expect(container).toMatchSnapshot();
  });
});
```

테스트를 실행하면 해당 파일 경로에 `__snapshot__` 디렉토리가 생성되고 .snap 확장자로 해당 테스트 당시HTML 코드가 문자열로 표시된다.

```bash
.
├── Article.test.tsx
├── ContractForm.test.tsx
├── Form.test.tsx
├── SignForm.test.tsx
└── __snapshots__
    └── Form.test.tsx.snap

##Form.test.tsx.samp
// Jest Snapshot v1, <https://goo.gl/fbAQLP>

exports[`Form 테스트 Snapshot: Form 컴포넌트가 렌더링 되면 이름이 표시된다. 1`] = `
<DocumentFragment>
  <form>
    <h2>
      Form
    </h2>
    <input
      type="text"
    />
    <button
      type="submit"
    >
      Submit
    </button>
  </form>
</DocumentFragment>
`;

```

이미 커밋된 `.snap` 파일과 비교하여 차이점이 발견되면 테스트가 실패한다.

```bash
  Form 테스트
    ✕ Snapshot: Form 컴포넌트가 렌더링 되면 이름이 표시된다. (11 ms)
    ○ skipped 이름을 표시한다.
    ○ skipped heading을 표시한다.

  ● Form 테스트 › Snapshot: Form 컴포넌트가 렌더링 되면 이름이 표시된다.

    expect(received).toMatchSnapshot()

    Snapshot name: `Form 테스트 Snapshot: Form 컴포넌트가 렌더링 되면 이름이 표시된다. 1`

    - Snapshot  - 2
    + Received  + 2

    @@ -1,6 +1,6 @@
    - <DocumentFragment>
    + <div>
        <form>
          <h2>
            Form
          </h2>
          <input
    @@ -10,6 +10,6 @@
            type="submit"
          >
            Submit
          </button>
        </form>
    - </DocumentFragment>
    + </div>

      15 |   it('Snapshot: Form 컴포넌트가 렌더링 되면 이름이 표시된다.', () => {
      16 |     const { container } = render(<Form name="Taro" />);
    > 17 |     expect(container).toMatchSnapshot();
         |                       ^
      18 |   });
      19 |
      20 |   //   it('"Submit" 버튼을 클릭하면 이벤트 핸들러가 실행된다..', () => {

      at Object.<anonymous> (tests/ui/Form.test.tsx:17:23)

 › 1 snapshot failed.
----------|---------|----------|---------|---------|-------------------
File      | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s
----------|---------|----------|---------|---------|-------------------
All files |   66.66 |      100 |      50 |      60 |
 Form.tsx |   66.66 |      100 |      50 |      60 | 5-6
----------|---------|----------|---------|---------|-------------------
Snapshot Summary
 › 1 snapshot failed from 1 test suite. Inspect your code changes or re-run jest with `-u` to update them.

Test Suites: 1 failed, 1 total
Tests:       1 failed, 2 skipped, 3 total
Snapshots:   1 failed, 1 total
```

만약 의도된 변경이라 테스트를 성공시키고 싶다면 `--updateSnapshot` 옵션을 추가해 jest를 실행하면 새로운 내용으로 갱신된다.

```bash
pnpm run jest --updateSnapshot
```

## 암묵적 역할과 접근 가능한 이름

테스팅 라이브러리가 가장 권장하는 '누구나 접근 가능한 쿼리' 중 대표적인 `getByRole`은 HTML 요소의 역할을 참조한다.

이는 W3C의 WAI-ARIA 스펙에 포함되어 있으며, HTML 태그에 `role` 속성으로 직접 지정할 수 있다. 또한 기본 HTML 태그들은 이미 암묵적인 역할이 미리 할당되어있다.

- ul, ol, dl → role=list
- li → role=listitem
- button → role=button

### aria 속성 값을 활용한 추출

```html
<h1 role="heading">대제목</h1>
<h2 role="heading">부제목</h2>
<h3 role="heading">소제목</h3>
```

```tsx
getByRole("heading", { level: 1 }); //h1 요소 추출
```

### 접근 가능한 이름을 활용한 추출

```html
<button>전송</button>
<button><img alt="전송" src="./전송-버튼-이미지.jpeg" /></button>
```

```tsx
getByRole("button", { name: "전송" });
```

### logRoles

테스팅 라이브러리의 `logRoles`함수를 사용하면 render 함수로 취득한 `container`를 인수로 전달하여 취득 가능한 요소들이 구분자와 함께 로그로 출력된다.

```tsx
import { logRoles } from "@testing-library/react";

test("logRoles: 렌더링 된 Form 엘리먼트의 역할을 출력한다.", () => {
  const { container } = render(<Form name="Taro" />);
  logRoles(container);
});
```

```bash
    heading:

    Name "Form":
    <h2 />

    --------------------------------------------------
    textbox:

    Name "":
    <input
      type="text"
    />

    --------------------------------------------------
    button:

    Name "Submit":
    <button
      type="submit"
    />

```

### 암묵적 역할 목록

| HTML 요소                 | WAI-ARIA의 암묵적 역할 | 참고                                |
| ------------------------- | ---------------------- | ----------------------------------- |
| `<article>`               | article                |                                     |
| `<aside>`                 | complementary          |                                     |
| `<nav>`                   | navigation             |                                     |
| `<header>`                | banner                 |                                     |
| `<footer>`                | banner                 |                                     |
| `<main>`                  | contentinfo            |                                     |
| `<section>`               | main                   |                                     |
| `<form>`                  | region                 | `aria-labelledby`가 지정된 경우     |
| `<button>`                | form                   | 접근 가능한 이름을 가진 경우로 한정 |
| `<a href="xxxxx">`        | link                   | `href`속성을 가진 경우로 한정       |
| `<input type="checkbox">` | checkbox               |                                     |
| `<input type="radio">`    | radio                  |                                     |
| `<input type="button">`   | button                 |                                     |
| `<input type="text">`     | textbox                |                                     |
| `<input type="password">` | 없음                   |                                     |
| `<input type="search">`   | searchbox              |                                     |
| `<input type="email">`    | textbox                |                                     |
| `<input type="url">`      | textbox                |                                     |
| `<input type="tel">`      | textbox                |                                     |
| `<input type="number">`   | spinbutton             |                                     |
| `<input type="range">`    | slider                 |                                     |
| `<select>`                | listbox                |                                     |
| `<optgroup>`              | group                  |                                     |
| `<option>`                | option                 |                                     |
| `<ul>`                    | list                   |                                     |
| `<ol>`                    | list                   |                                     |
| `<li>`                    | listitem               |                                     |
| `<table>`                 | table                  |                                     |
| `<caption>`               | caption                |                                     |
| `<th>`                    | columnheader/rowheader | 행/열 여부에 따라 달라짐            |
| `<td>`                    | cell                   |                                     |
| `<tr>`                    | row                    |                                     |
| `<fieldset>`              | group                  |                                     |
| `<legend>`                | 없음                   |                                     |
