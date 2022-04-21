## Typescript 类型积累

> React 组件类型梳理







## Button

**types.ts** 

~~~js
import { ButtonHTMLAttributes, AnchorHTMLAttributes } from 'react'

export type ButtonType = 'default' | 'primary' | 'link' | 'icon' | 'link-default' | 'default-link' | 'link-primary' | 'primary-link' // 共 8 种类型按钮
export type ButtonSize = 'small' | 'normal' | 'large'   // 共 3 中大小
export type ButtonTheme = 'light' | 'dark' // 浅色 / 深色模式
export type WithArrow = 'enable' | 'disable' | 'pc-only' | 'm-only' // pc & m 都有 / pc & m 都没有 / 仅在 pc 有 / 仅在 m 有
export type Highlight = 'enable' | 'disable' | 'pc-only' | 'm-only' // pc & m 都高亮 / pc & m 都不高亮 / 仅在 pc 高亮 / 仅在 m 高亮

interface BaseButtonProps {
  btnType?: ButtonType // type 为 <button> 的自带属性，所以这里使用 btnType 作为属性名
  withArrow?: WithArrow // 不传默认为不带箭头
  size?: ButtonSize
  btnTheme?: ButtonTheme
  className?: string
  disabled?: boolean
  highlight?: Highlight // 不传默认为不高亮
  children?: React.ReactNode
}

type AllButtonProps = BaseButtonProps & ButtonHTMLAttributes<HTMLElement>
type AllAnchorProps = BaseButtonProps & AnchorHTMLAttributes<HTMLElement>

export type ButtonProps = Partial<AllButtonProps & AllAnchorProps>

~~~











## Input 输入框

