# React-styling

생성일: 2021년 8월 30일 오후 2:06

# 1. sass

- sass(Syntactically Awesome style sheets)는 CSS pre-processor로서, 복잡한 작업을 쉽게 할 수 있게 해주고, 코드의 재활용성을 높여줄 뿐만 아니라 코드의 가독성을 높여주어 유지보수를 쉽게 해준다.

## Setting

```jsx
// create new react-app
npx create-react-app styling

cd styling-with-sass
npm install node-sass
```

## Button Component 만들기

### components/Button.js

```jsx
import React from 'react'
import './Button.scss'

function Button({children}) {
	return <button className="Button">{children}</button>
}

export default Button
```

### components/Button.scss

```scss
$blue: #228be6

.Button {
	display: inline-flex
	color: white;
	font-weight: bold
	outline: none
	border-radius: 4px;
	border: none
	cursor: pointer

	height: 2.25rem;
	padding-left: 1rem;
	padding-right: 1rem
	font-size: 1rem

	background: $blue
	&:hover {
		background: lighten($blue, 10%)
	}

	&:active {
		background: darken($blue, 10%)
	}
}
```

### Button 사이즈 조정하기

- 조건부로 CSS 클래스 넣기.

```scss
npm install classnames
```

### components/Button.js

```scss
import React from 'react';
import './Button.scss';

function Button({ children, size }) {
  //return <button className={['Button', size].join(' ')}>{children}</button>;
	return <button className={classNames('Button', size)}>{children}</button>
}

Button.defaultProps = {
	size: 'medium'
}
	
export default Button;
```

### components/Button.scss

```scss
$blue: #228be6;

.Button {
  display: inline-flex;
  color: white;
  font-weight: bold;
  outline: none;
  border-radius: 4px;
  border: none;
  cursor: pointer;

  // 사이즈 관리
  &.large {  // same as .Button.larg
    height: 3rem;
    padding-left: 1rem;
    padding-right: 1rem;
    font-size: 1.25rem;
  }

  &.medium {
    height: 2.25rem;
    padding-left: 1rem;
    padding-right: 1rem;
    font-size: 1rem;
  }

  &.small {
    height: 1.75rem;
    font-size: 0.875rem;
    padding-left: 1rem;
    padding-right: 1rem;
  }

  background: $blue;
  &:hover {
    background: lighten($blue, 10%);
  }

  &:active {
    background: darken($blue, 10%);
  }

	& + & { // .Button + .Button 버튼이 함께 있는 경우 우측에 여백설정.
		margin-left: 1rem 
	}
}
```

### Button 색상 설정하기.

> 개발을 할 때 색상을 설정할 때 [open-color](https://yeun.github.io/open-color/)를 사용

### components/Button.js

```scss
import React from 'react';
import classNames from 'classnames';
import './Button.scss';

function Button({ children, size, color }) {
  return (
    <button className={classNames('Button', size, color)}>{children}</button>
  );
}

Button.defaultProps = {
  size: 'medium',
  color: 'blue'
};

export default Button;
```

### components/Button.scss

```scss
//--- below the code that we write before
$grey: #495057
$pink: #f06595

&.blue {
    background: $blue;
    &:hover {
      background: lighten($blue, 10%);
    }

    &:active {
      background: darken($blue, 10%);
    }
  }

  &.gray {
    background: $gray;
    &:hover {
      background: lighten($gray, 10%);
    }

    &:active {
      background: darken($gray, 10%);
    }
  }

	&.pink {
		background: $pink
		&:hover {
			background: lighten($pink, 10%)
		}

		&:active {
			background: darken($pink, 10%)
		}
	}
```

### mixin으로 반복되는 코드 재사용

### components/Button.scss

```scss
@mixin button-color($color) {
	background: $color;
	
	&:hover: {
		background: lighten($color, 10%)
	}

	&:active {
		background: darken($color, 10%)
	}
}

&.blue {
	@include button-color($blue)
}

//... and so on
```

### 최종 코드

### components/Button.js

```scss
import React from 'react';
import classNames from 'classnames';
import './Button.scss';

function Button({children, size, color, outline, fullwidth, ...rest}) {
	return (
//outline, fullwidth는 true값일 때만 동작.
		<button = {classNames('button', size, color, {outline, fullwidth})}
		{...rest}>
			{children}
		</button>
	)
}

Button.defaultProps = {
	size: 'medium',
	color: 'blue'
}

export default Button
```

### components/Button.scss

```scss
$blue: #228be6;
$gray: #495057;
$pink: #f06595;

@mixin button-color($color) {
  background: $color;
  &:hover {
    background: lighten($color, 10%);
  }
  &:active {
    background: darken($color, 10%);
  }
  &.outline {
    color: $color;
    background: none;
    border: 1px solid $color;
    &:hover {
      background: $color;
      color: white;
    }
  }
}

.Button {
  display: inline-flex;
  color: white;
  font-weight: bold;
  outline: none;
  border-radius: 4px;
  border: none;
  cursor: pointer;

  // 사이즈 관리
  &.large {
    height: 3rem;
    padding-left: 1rem;
    padding-right: 1rem;
    font-size: 1.25rem;
  }

  &.medium {
    height: 2.25rem;
    padding-left: 1rem;
    padding-right: 1rem;
    font-size: 1rem;
  }

  &.small {
    height: 1.75rem;
    font-size: 0.875rem;
    padding-left: 1rem;
    padding-right: 1rem;
  }

  // 색상 관리
  &.blue {
    @include button-color($blue);
  }

  &.gray {
    @include button-color($gray);
  }

  &.pink {
    @include button-color($pink);
  }

  & + & {
    margin-left: 1rem;
  }

  &.fullWidth {
    width: 100%;
    justify-content: center;
    & + & {
      margin-left: 0;
      margin-top: 1rem;
    }
  }
}
```

### App.js

```jsx
import React from 'react';
import './App.scss';
import Button from './components/Button';

function App() {
  return (
    <div className="App">
      <div className="buttons">
        <Button size="large" onClick={() => console.log('클릭됐다!')}>
          BUTTON
        </Button>
        <Button>BUTTON</Button>
        <Button size="small">BUTTON</Button>
      </div>
      <div className="buttons">
        <Button size="large" color="gray">
          BUTTON
        </Button>
        <Button color="gray">BUTTON</Button>
        <Button size="small" color="gray">
          BUTTON
        </Button>
      </div>
      <div className="buttons">
        <Button size="large" color="pink">
          BUTTON
        </Button>
        <Button color="pink">BUTTON</Button>
        <Button size="small" color="pink">
          BUTTON
        </Button>
      </div>
      <div className="buttons">
        <Button size="large" color="blue" outline>
          BUTTON
        </Button>
        <Button color="gray" outline>
          BUTTON
        </Button>
        <Button size="small" color="pink" outline>
          BUTTON
        </Button>
      </div>
      <div className="buttons">
        <Button size="large" fullWidth>
          BUTTON
        </Button>
        <Button size="large" color="gray" fullWidth>
          BUTTON
        </Button>
        <Button size="large" color="pink" fullWidth>
          BUTTON
        </Button>
      </div>
    </div>
  );
}
```

## 결과물

![Untitled](React-styling%208681f27a8fe64bc59abaf4a7403b7fb8/Untitled.png)

---

# 2. CSS module

- 리엑트 프로젝트에서 컴포넌트를 스타일링 할 때 css module을 사용하여  CSS 클래스 중첩 방지.
- CRA로 만든 프로젝트에서 CSS module를 사용할 때에는, css 파일의 확장자를 .module.css로 하면 된다

### CSS 클래스 네이밍 규칙

- 컴포넌트의 이름은 다른 컴포넌트랑 중복되지 않게 한다.
- 컴포넌트의 최상단 CSS 클래스는 컴포넌트의 이름과 일치시킨다.
- 컴포넌트 내부에서 보여지는 CSS클래스는 CSS selector를 잘 활용한다.

### Example

### components/CheckBox.js

```jsx
import React from 'react';
import { MdCheckBox, MdCheckBoxOutlineBlank } from 'react-icons/md';
import styles from './CheckBox.module.css';
import classNames from 'classnames/bind';

const cx = classNames.bind(styles);

function CheckBox({ children, checked, ...rest }) {
  return (
    <div className={cx('checkbox')}>
      <label>
        <input type="checkbox" checked={checked} {...rest} />
        <div className={cx('icon')}>
          {checked ? (
            <MdCheckBox className={cx('checked')} />
          ) : (
            <MdCheckBoxOutlineBlank />
          )}
        </div>
      </label>
      <span>{children}</span>
    </div>
  );
}

export default CheckBox;
```

> Font Awesome, Material Design Icons 이용 시 [참조](https://react-icons.github.io/react-icons/#/)

```jsx
npm install react-icons
```

### components/CheckBox.module.css

```jsx
.checkbox {
  display: flex;
  align-items: center;
}

.checkbox label {
  cursor: pointer;
}

/* 실제 input 을 숨기기 위한 코드 */
.checkbox input {
  width: 0;
  height: 0;
  position: absolute;
  opacity: 0;
}

.checkbox span {
  font-size: 1.125rem;
  font-weight: bold;
}

.icon {
  display: flex;
  align-items: center;
  /* 아이콘의 크기는 폰트 사이즈로 조정 가능 */
  font-size: 2rem;
  margin-right: 0.25rem;
  color: #adb5bd;
}

.checked {
  color: #339af0;
}
```

### App.js

```jsx
import CheckBox from './components/CheckBox';

function App() {
  const [check, setCheck] = useState(false);
  const onChange = e => {
    setCheck(e.target.checked);
  };
  return (
    <div>
      <CheckBox onChange={onChange} checked={check}>
        다음 약관에 모두 동의
      </CheckBox>
      <p>
        <b>check: </b>
        {check ? 'true' : 'false'}
      </p>
    </div>
  );
}

export default App;
```

### 결과물

![Untitled](React-styling%208681f27a8fe64bc59abaf4a7403b7fb8/Untitled%201.png)

---

## 3. Styled-components

## Tagged Template Literal

```jsx
function sample(texts, ...fns) {
	const mockProps = {
		title: 'Hello',
		body: 'world'
	}

	return texts.reduce((result, text, i) => `${result}${text}${fns[i] ? fns[i](mockProps) : ''}`, '')
}

sample`
  제목: ${props => props.title}
  내용: ${props => props.body}
`
```

### Styled-components 사용하기

```jsx
npx create-react-app styled-components
cd styled-components
npm install styled-components
```

### App.js

```jsx
import React from 'react'
import styled, {css} from 'styled-components'

const Circle = styled.div`
	width: 5rem;
	height: 5rem;
	background: {props => props.color || 'black'}
	border-radius: 50%
	${props => 
		props.huge && 
		css`
			width: 10rem
			height: 10rem
		`}
`

function App() {
	return <Circle color="red" huge />
}
```

### Polished의 스타일 관련 유틸 함수 사용하기

```jsx
npm install polished
```

### Button.js

```jsx
import React from 'react';
import styled from 'styled-components';
import {darken, lighten} from 'polished'

const StyledButton = styled.Button`
	// same as before

	&:hover {
		background: ${lighten(0.1, '#228be6')}
	}
`
```

### ThemeProvider 사용하기

### App.js

```jsx
import React from 'react';
import styled, { ThemeProvider } from 'styled-components';
import Button from './components/Button';

const AppBlock = styled.div`
  width: 512px;
  margin: 0 auto;
  margin-top: 4rem;
  border: 1px solid black;
  padding: 1rem;
`;

function App() {
  return (
    <ThemeProvider
      theme={{
        palette: {
          blue: '#228be6',
          gray: '#495057',
          pink: '#f06595'
        }
      }}
    >
      <AppBlock>
        <Button>BUTTON</Button>
      </AppBlock>
    </ThemeProvider>
  );
}

export default App;
```

### Button.js

```jsx
import React from 'react';
import styled, { css } from 'styled-components';
import { darken, lighten } from 'polished';

const StyledButton = styled.button`
//색상
${props => {
	const selected = props.theme.palette[props.color];

//same as 
//${({ theme, color }) => {
//    const selected = theme.palette[color];

	return css`
		background: ${selected}
		&:hover {
			background: lighten(0.1, selected)
		}
	`
}}
`
function Button({ children, ...rest }) {
  return <StyledButton {...rest}>{children}</StyledButton>;
}

Button.defaultProps = {
  color: 'blue'
};

export default Button;

```

### 여러가지 props로 Button styling 하기.

```jsx
import React from 'react';
import styled, { css } from 'styled-components';
import { darken, lighten } from 'polished';

const colorStyles = css`
  ${({ theme, color }) => {
    const selected = theme.palette[color];
    return css`
      background: ${selected};
      &:hover {
        background: ${lighten(0.1, selected)};
      }
      &:active {
        background: ${darken(0.1, selected)};
      }
      ${props =>
        props.outline &&
        css`
          color: ${selected};
          background: none;
          border: 1px solid ${selected};
          &:hover {
            background: ${selected};
            color: white;
          }
        `}
    `;
  }}
`;

const sizes = {
  large: {
    height: '3rem',
    fontSize: '1.25rem'
  },
  medium: {
    height: '2.25rem',
    fontSize: '1rem'
  },
  small: {
    height: '1.75rem',
    fontSize: '0.875rem'
  }
};

const sizeStyles = css`
  ${({ size }) => css`
    height: ${sizes[size].height};
    font-size: ${sizes[size].fontSize};
  `}
`;

const fullWidthStyle = css`
  ${props =>
    props.fullWidth &&
    css`
      width: 100%;
      justify-content: center;
      & + & {
        margin-left: 0;
        margin-top: 1rem;
      }
    `}
`;

const StyledButton = styled.button`
  /* 공통 스타일 */
  display: inline-flex;
  outline: none;
  border: none;
  border-radius: 4px;
  color: white;
  font-weight: bold;
  cursor: pointer;
  padding-left: 1rem;
  padding-right: 1rem;

  /* 크기 */
  ${sizeStyles}

  /* 색상 */
  ${colorStyles}

  /* 기타 */
  & + & {
    margin-left: 1rem;
  }

  ${fullWidthStyle}
`;

function Button({ children, color, size, outline, fullWidth, ...rest }) {
  return (
    <StyledButton
      color={color}
      size={size}
      outline={outline}
      fullWidth={fullWidth}
      {...rest}
    >
      {children}
    </StyledButton>
  );
}

Button.defaultProps = {
  color: 'blue',
  size: 'medium'
};

export default Button;
```

### keyframes 유틸을 이용해서 트랜지션 구현하기

```jsx
import React, { useState, useEffect } from 'react';
import styled, { keyframes, css } from 'styled-components';
import Button from './Button';

const fadeIn = keyframes`
  from {
    opacity: 0
  }
  to {
    opacity: 1
  }
`;

const fadeOut = keyframes`
  from {
    opacity: 1
  }
  to {
    opacity: 0
  }
`;

const slideUp = keyframes`
  from {
    transform: translateY(200px);
  }
  to {
    transform: translateY(0px);
  }
`;

const slideDown = keyframes`
  from {
    transform: translateY(0px);
  }
  to {
    transform: translateY(200px);
  }
`;

const DarkBackground = styled.div`
  position: fixed;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(0, 0, 0, 0.8);

  animation-duration: 0.25s;
  animation-timing-function: ease-out;
  animation-name: ${fadeIn};
  animation-fill-mode: forwards;

  ${props =>
    props.disappear &&
    css`
      animation-name: ${fadeOut};
    `}
`;

const DialogBlock = styled.div`
  width: 320px;
  padding: 1.5rem;
  background: white;
  border-radius: 2px;
  h3 {
    margin: 0;
    font-size: 1.5rem;
  }
  p {
    font-size: 1.125rem;
  }

  animation-duration: 0.25s;
  animation-timing-function: ease-out;
  animation-name: ${slideUp};
  animation-fill-mode: forwards;

  ${props =>
    props.disappear &&
    css`
      animation-name: ${slideDown};
    `}
`;

const ButtonGroup = styled.div`
  margin-top: 3rem;
  display: flex;
  justify-content: flex-end;
`;

const ShortMarginButton = styled(Button)`
  & + & {
    margin-left: 0.5rem;
  }
`;

function Dialog({
  title,
  children,
  confirmText,
  cancelText,
  onConfirm,
  onCancel,
  visible
}) {
  const [animate, setAnimate] = useState(false);
  const [localVisible, setLocalVisible] = useState(visible);

  useEffect(() => {
    // visible 값이 true -> false 가 되는 것을 감지
    if (localVisible && !visible) {
      setAnimate(true);
      setTimeout(() => setAnimate(false), 250);
    }
    setLocalVisible(visible);
  }, [localVisible, visible]);

  if (!animate && !localVisible) return null;
  return (
    <DarkBackground disappear={!visible}>
      <DialogBlock disappear={!visible}>
        <h3>{title}</h3>
        <p>{children}</p>
        <ButtonGroup>
          <ShortMarginButton color="gray" onClick={onCancel}>
            {cancelText}
          </ShortMarginButton>
          <ShortMarginButton color="pink" onClick={onConfirm}>
            {confirmText}
          </ShortMarginButton>
        </ButtonGroup>
      </DialogBlock>
    </DarkBackground>
  );
}

Dialog.defaultProps = {
  confirmText: '확인',
  cancelText: '취소'
};

export default Dialog;
```