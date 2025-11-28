# Redux Toolkit

## createAsyncThunk

> 用于简化异步请求逻辑
> 会自动生成三个 `Action` : `pending` `fulfilled` `rejected`

- pending
  - 异步函数开始执行时触发
- fulfilled
  - 异步函数成功时触发
- rejected
  - 异步函数失败时触发

- 实例模板
  - `Slice`

    ```javaScript
    import { createSlice, createAsyncThunk } from '@reduxjs/toolkit'
    import api from '@/api/user'   // 假设你的接口文件

    // ----------- 异步请求（模板）-----------
    export const fetchUser = createAsyncThunk(
      'user/fetchUser',
      async (id, { rejectWithValue }) => {
        try {
          const res = await api.getUser(id)
          return res.data  // 成功
        } catch (err) {
          return rejectWithValue(err.response?.data || '请求失败') // 失败
        }
      }
    )

    const initialState = {
      profile: null,
      loading: false,
      error: null,
    }

    // ----------- Slice（模板） ----------
    const userSlice = createSlice({
      <!-- sliceName -->
      name: 'user', 
      <!-- 初始化状态 -->
      initialState,
      <!-- 导出自定义方法 -->
      reducers: {
        // 同步修改（模板）
        updateProfile(state, action) {
          state.profile = { ...state.profile, ...action.payload }
        },

        resetUser(state) {
          state.profile = null
          state.error = null
          state.loading = false
        },
      },

      // ----------- 预处理异步请求的三种状态 ----------
      extraReducers: (builder) => {
        builder
          .addCase(fetchUser.pending, (state) => {
            state.loading = true
            state.error = null
          })
          .addCase(fetchUser.fulfilled, (state, action) => {
            state.loading = false
            state.profile = action.payload
          })
          .addCase(fetchUser.rejected, (state, action) => {
            state.loading = false
            state.error = action.payload
          })
      },
    })

    export const { updateProfile, resetUser } = userSlice.actions

    ```

  - `store`
    - 导出redux到全局可引用

    ```javaScript
    import { configureStore } from '@reduxjs/toolkit'
    import userReducer from './userSlice'

    export const store = configureStore({
      reducer: {
        user: userReducer,
      },
    })

    ```
  
  - view

    ```javaScript
    import { useDispatch, useSelector } from 'react-redux'
     <!-- 引用redux 方法 -->
    import { fetchUser, updateProfile } from '@/store/userSlice'

    export default function UserPage() {
      const dispatch = useDispatch()

      <!-- 引用redux状态值 -->
      const { profile, loading, error } = useSelector(state => state.user)

      return (
        <div>
          模板
        </div>
      )
    }
    ```
