## 스토리북 기초

스토리북은 UI 컴포넌트 탐색기의 역할을 제공하지만, 테스트 기능도 제공한다.

### 설치 방법

```bash
npx storybook init
```

### 기본 사용법

**스토리 파일**

`.stories.tsx|jsx` 형태로 스토리 파일을 추가할 수 있다. 해당 스토리의 정보를 담는 메타 객체를 반드시 export default로 내보내야 한다.

```tsx
import type { Meta, StoryObj } from '@storybook/react';
import { fn } from '@storybook/test';

import { Button } from './Button';

// More on how to set up stories at: <https://storybook.js.org/docs/writing-stories#default-export>
const meta = {
  title: 'Example/Button',
  component: Button,
  parameters: {
    // Optional parameter to center the component in the Canvas. More info: <https://storybook.js.org/docs/configure/story-layout>
    layout: 'centered',
  },
  // This component will have an automatically generated Autodocs entry: <https://storybook.js.org/docs/writing-docs/autodocs>
  tags: ['autodocs'],
  // More on argTypes: <https://storybook.js.org/docs/api/argtypes>
  argTypes: {
    backgroundColor: { control: 'color' },
  },
  // Use `fn` to spy on the onClick arg, which will appear in the actions panel once invoked: <https://storybook.js.org/docs/essentials/actions#action-args>
  args: { onClick: fn() },
} satisfies Meta<typeof Button>;

export default meta;
type Story = StoryObj<typeof meta>;

// More on writing stories with args: <https://storybook.js.org/docs/writing-stories/args>
export const Primary: Story = {
  args: {
    primary: true,
    label: 'Button',
  },
};

export const Secondary: Story = {
  args: {
    label: 'Button',
  },
};

export const Large: Story = {
  args: {
    size: 'large',
    label: 'Button',
  },
};

export const Small: Story = {
  args: {
    size: 'small',
    label: 'Button',
  },
};

```

**개별 스토리 등록하기**

스토리 파일 내에서 다른 이름으로 export하면 서로 다른 스토리로 등록됩니다. 내보내는 이름을 기준으로 스토리북 화면 사이드 탭에 생성된다.

```tsx
export const Primary: Story = {
  args: {
    primary: true,
    label: 'Button',
  },
};

export const Secondary: Story = {
  args: {
    label: 'Button',
  },
};

export const Large: Story = {
  args: {
    size: 'large',
    label: 'Button',
  },
};

export const Small: Story = {
  args: {
    size: 'small',
    label: 'Button',
  },
};

```

### 스토리북 계층

모든 스토리에서는 깊은 병합 형식으로 적용되는데 `Global` `Component` `Story` 라는 세 단계의 설정으로 적용할 수 있다.

* **Global:** 모든 스토리에 전역적으로 적용할 설정이다. (.storybook/preview.ts)
* **Component:** 스토리 파일에 개별적으로 적용할 설정이다. (storyname.stories.tsx|jsx)
* **Story:** 개별 스토리에 적용할 각각의 객체이다. (export const)

## 필수 애드온

### @storybook/addon-essentials

기본적으로 스토리북을 설치하면 함께 설치 되는 필수 애드온이다. 대표적으로 `@storybook/addon-controls`라는 모듈에서 제공하는데, 해당 패키지는 `addon-essntials` 패키지에 포함되어 있다.

### @storybook/addon-actions

UI 컴포넌트에서 내부 로직을 처리하기 위해 Props로 전달받은 이벤트 핸들러를 호출하는데, 어떻게 호출됐는지 로그를 출력해주는 기능이다.

```tsx
export const parameters = {
	//on으로 시작하는 모든 이벤트 핸들러 자동적으로 패널에 로깅
  actions: { argTypesRegex: "^on[A-Z].*" },
  controls: {
    matchers: {
      color: /(background|color)$/i,
      date: /Date$/,
    },
  },
  nextRouter: {
    Provider: RouterContext.Provider,
  },
  viewport: {
    viewports: INITIAL_VIEWPORTS,
  },
  msw: { handlers: [handleGetMyProfile()] },
  layout: "fullscreen",
};

```

### @storybook/addon-viewport

반응형 컴포넌트를 확인하기 위한 애드온이다.

```tsx
import { Preview } from '@storybook/your-renderer';
import { INITIAL_VIEWPORTS } from '@storybook/addon-viewport';
 
const preview: Preview = {
  parameters: {
    viewport: {
      viewports: INITIAL_VIEWPORTS,
      defaultViewport: 'ipad',
    },
  },
};
 
export default preview;
```

## Context API 의존 스토리

React의 Context API에 의존하는 스토리는 스토리북의 `Decorator`를 활용하는 것이 좋다. 예를 들어 UI 컴포넌트 외부에 어떠한 스타일을 추가하고 싶다면 아래와 같이 `decorators` 배열에 함수를 추가한다.

```tsx
const meta: Meta<typeof Tab> = {
	//..기존 스토리 정보
	decorators: [
    Story => (
      <div className="flex flex-col gap-2 justify-center items-center">
        <Story />
      </div>
    )
  ],
}
```

해당 계층은 Storybook Component 단계이므로 모든 스토리에 동일한 스타일이 적용된다.

## 웹 API에 의존하는 스토리

웹 API 통신에 의존하는 컴포넌트는 스토리에도 웹 API가 필요하며, 이를 위해 모킹 서버가 필요하다. (MSW)

```bash
npm i msw msw-storybook-addon --save-dev
```

필요한 애드온과 msw를 설치한 다음, 아래와 같이 핸들러를 등록한다.

```tsx
//storybook/preview.js
export const parameters = {
  // 기타 설정 생략
  msw: {
    handlers: [
      rest.get("/api/my/profile", async (_, res, ctx) => {
        return res(
          ctx.status(200),
          ctx.json({
            id: 1,
            name: "EonsuBae",
            bio: "프런트엔드 엔지니어. 타입스크립트와 UI 컴포넌트 테스트에 관심이 있습니다.",
            twitterAccount: "eonsu-bae",
            githubAccount: "eonsu-bae",
            imageUrl: "/__mocks__/images/img01.jpg",
            email: "eonsubae@example.com",
            likeCount: 1,
          })
        );
      }),
    ],
  },
};
```

요청 핸들러가 스토리에 적용되는 우선순위는 다음과 같다.

1. Story
2. Component
3. Global (preview)

만일 최하위 스토리 단계에서 개별적으로 적용하려면 아래와 같이 파라미터 정보에 등록하면 된다.

```tsx
export const NotLoggedIn: Story = {
	parameters: {
		msw: {
			handlers: [
				rest.get("/api/my/profile", async (_, res, ctx) => {
					return res(ctx.status(401));
				});
			],
		},
	},
}
```

## Next Router에 의존하는 스토리

특정 URL에서 사용할 수 있는 컴포넌트를 테스트하려면 라우팅 정보가 필요하다. 이를 확인하기 위해 `storybook-addon-next-router` 애드온을 사용할 수 있다.

```bash
npm i storybook-addon-next-router
```

```tsx
//./storybook/main.js
module.exports = {
	stories:["../src/**/*.stories.@(js|jsx|ts|tsx)"],
	addons:["storybook-addon-nextrouter"]
}
```

아래와 같이 각 스토리에 테스팅할 라우팅 경로를 설정한다.

```tsx
export const RouteMyPosts: Story ={
	parameters: {
		nextRouter: { pathname: "/my/posts" },
	},
}

export const RouteMyPostsCreate: Story ={
	parameters: {
		nextRouter: { pathname: "/my/posts/create" },
	},
}
```

## 인터랙션 스토리 테스트

Form 유효성 검사나 UI 테스트 등 사용자 인터랙션이 필요한 경우, `play` 함수를 통해 스토리상에서 인터랙션이 필요한 UI를 노출시킬 수 있다.

```bash
npm i @storybook/testing-library @storytbook/jest @storybook/addon-interactions
```

```tsx
//./storybook/main.ts
import path from 'path';
import type { StorybookConfig } from '@storybook/react-webpack5';

const config: StorybookConfig = {
  stories: ['../src/stories/*.stories.@(js|jsx|ts|tsx)'],
  addons: [
		//..기존 addon
		"@storybook/addon-interactions"
		],
	features: {
		interactionsDebugger: true
	}  
};
export default config;

```

아래와 같이 Jest 테스트 코드 처럼, `userEvent` API 를 활용해 UI 컴포넌트에 인터랙션을 할당할 수 있다.

```tsx
import 'react-device-frameset/styles/marvel-devices.min.css';
import React from 'react';
import { useState } from 'react';
import { Canvas, Subtitle, Title } from '@storybook/blocks';
import { DocSource } from './components/DocSource';
import { Tab, Text } from '../ui';
import type { Meta } from '@storybook/react';

const meta: Meta<typeof Tab> = {
  title: 'Action/Tab',
  component: Tab,
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);
    await userEvent.click(canvas.getByText('Tab1'));
  },
 };
export default meta;

```

인터랙션을 제대로 할당하지 않으면 테스트가 중단되고 패널에 ‘FAIL’이라는 경고가 노출된다.

## 웹 접근성 스토리 테스트

`@storybook/addon-a11y` 애드온을 설치하면 스토리북 탐색기에서 접근성 관련 경고를 알려준다.

```bash
npm i @storybook/addon-a11y -D
```

```tsx
import path from 'path';
import type { StorybookConfig } from '@storybook/react-webpack5';

const config: StorybookConfig = {
  stories: ['../src/stories/*.stories.@(js|jsx|ts|tsx)'],
  staticDirs: [{ from: '../src/assets', to: '/' }],
  addons: [
    '@storybook/addon-a11y',
  ]
};
export default config;

```

### 규칙 완화하기

팀 내에서 어느정도 합의된 규칙은 개별 스토리에서도 `rules` 속성을 통해 개별 규칙을 무효화시킬 수 있다.

```tsx
import 'react-device-frameset/styles/marvel-devices.min.css';
import React from 'react';
import { useState } from 'react';
import { Canvas, Subtitle, Title } from '@storybook/blocks';
import { DocSource } from './components/DocSource';
import { Tab, Text } from '../ui';
import type { Meta } from '@storybook/react';

const meta: Meta<typeof Tab> = {
  title: 'Action/Tab',
  component: Tab,
  tags: ['autodocs'],
  decorators: [
    Story => (
      <div className="flex flex-col gap-2 justify-center items-center">
        <Story />
      </div>
    )
  ],
  argTypes: {
    className: {
      control: 'text',
      description: 'Tab 컴포넌트에 추가할 클래스 이름을 설정합니다.'
    },
    variant: {
      control: 'inline-radio',
      options: ['standard', 'compact', 'box'],
      description: 'Tab 컴포넌트의 스타일을 설정합니다. (기본값: standard)',
      defaultValue: 'standard'
    },
    onChange: {
      control: 'object',
      description: 'Tab 컴포넌트의 선택된 탭이 변경될 때 실행될 함수를 설정합니다.'
    },
    defaultIndex: {
      control: 'number',
      description: 'Tab 컴포넌트의 초기 탭을 설정합니다.'
    },
    hasMargin: {
      control: 'boolean',
      description: 'Tab 컴포넌트의 좌우 여백 여부를 설정합니다. (responsive가 mobile일 때 적용됩니다.)'
    }
  },
  parameters: {
    a11y: {
      config: {
        rules: [
          {
            name: 'color-contrast',
            enabled: false
          }
        ]
      }
    },
	 ...
};

```

## 스토리북 테스트 러너

스토리북의 테스트 러너는 스토리를 실행가능한 테스트로 변환하는 애드온이다. 이를 사용해 기존에 사용하고 있는 Jest, Playwright에서 사용할 수 있다.

```bash
npm install @storybook/test-runner -D 
```

```json
//package.json
{
	"scripts": {
		"test:storybook": "test-storybook"
	}
}
```
