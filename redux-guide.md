## Структура папки redux

1. Файл reduxStore.ts
1. Папка с редюсерами, в которой хранятся файлы - имяReducer.ts
1. Папка actions, в которой хранится файл actions.ts
1. Папка thunks, в которой хранится файл thunks.ts
1. В папке redux можно создать папку types, где можно размещать глобальные для файлов redux типы

## Правила оформления redux файлов (с простыми примерами кода)


### reduxStore.ts


#### rootReducer

Константа rootReducer принимает state и возвращает новый state

```
const rootReducers = combineReducers({
    profilePage: profileReducer,
    dialogsPage: dialogsReducer,
    sidebar: sidebarReducer,
    usersPage: userReducer,
    auth: authReducer,
    form: formReducer,
    app: appReducer
});
```

Типизация:

1. rootReducer
``` 
type RootReducerType = typeof rootReducers; 
export type AppStateType = ReturnType<RootReducerType>;
```
1. Здесь же делаем общий тпи для всех actions  в приложении
```
export type InferActionsType<T> = T extends {[key: string]: (...args: any[]) => infer U} ? U : never;
```
1. Здесь же типизация для всех thunks в приложении
```export type BaseThunkType<A extends Action, R = Promise<void>> = ThunkAction<R, AppStateType, unknown, A>;
```

#### reducer.ts

В файлах данного типа хранятся initialState и сам reducer

Типизация:

1. initialState
```
let initialState = {
    users: [] as UserType[];
    pageSize: 10;
    isFetching: false;
    profile: null as ProfileType | null;
    status: '';
};
```
Будет использоваться в типизации reducer
```
export type initialStateType = typeof initialState;
```
1. Импортируем из reduxStore InferActionsType для типизации actions, необходимых для данного reducer 
```
type ActionsType = InferActionsType<typeof имяActions>;
```
1. Типизируем reducer
```
const usersReducer = (state = initialState, action: ActionsTypes): InitialStateType => {
    switch (action.type) {
        case SET_USERS:
            return { ...state, users: action.users };
        default:
            return state;
    };
};
```

#### actions.ts

Имеют следующий вид:
```
export const actions = {
    addPost: (newPostBody: string) => ({type: ADD_POST, newPostBody} as const),
    deletePost: (postId: number) => ({type: DELETE_POST, postId} as const),
    setUserProfile: (profile: ProfileType) => ({type: SET_USER_PROFILE, profile} as const),
    setStatus: (status: string) => ({type: SET_STATUS, status} as const),
    savePhotoSuccess: (photos: PhotosType) => ({type: SAVE_PHOTO_SUCCESS, photos} as const)
};
```

Типизация:

Описана в пункте reducer.ts

#### thunks.ts

Используем код с async await:
```
export const getUserProfileThC = (userId: number): ThunkType => {
    return async (dispatch) => {
        let data = await profileAPI.getUserProfile(userId);
        dispatch(actions.setUserProfile(data));
    };
};
```
В имени не обязательно в конце добавлять ThC - ThunkCreator (можно использовать для удобства)

Типизация:

1. Импортируем из reduxStore BaseThunkType для типизации thunks:
```
type ThunkType = ThunkAction<Promise<void>, AppStateType, unknown, ActionsTypes>;
```
В данном фрагменте описываем thunk:
```
<Promise<void>, AppStateType, unknown, ActionsTypes>
```
- Promise<void> - всегда возвращает promise
- AppStateType - типизированный state из redduxStore.ts
- unknown - ...
- ActionsTypes - типизированные actions для данного случая


## compose & connect

Данный функционал используется в контейнерных компонентах, но относится он к redux.

#### compose

Типизация:

Приписываем к compose React.ComponentType:
```
export default compose<React.ComponentType>(
    connect<MapStatePropsType, MapDispatchPropsType, {} ,AppStateType>(mapStateToProps, { addMessage: actions.addMessage }),
    withAuthRedirect
)(Dialogs);
```

#### connect

Типизация:

```
export default connect<MapStatePropsType, MapDispatchPropsType, OwnPropsType, AppStateType>(mapStateToProps, {
    getUsersThunkCreator,
    followThunkCreator,
    unfollowThunkCreator
})(UsersContainer);
```

- MapStatePropsType - типизированные данные, получаемые контейнерной компонентой

Например:
Принятые данные
```
const mapStateToProps = (state: AppStateType): MapStatePropsType => {
    return {
        users: getUsers(state),
        pageSize: getPageSize(state),
        totalUsersCount: getTotalUsersCount(state),
        currentPage: getCurrentPage(state),
        isFetching: getIsFetching(state),
        followingInProgress: getFollowingInProgress(state),
        portionSize: state.usersPage.portionSize
    };
};
```
Их типизация
```
type MapStatePropsType = {
    currentPage: number,
    pageSize: number,
    isFetching: boolean,
    totalUsersCount: number,
    portionSize: number,
    users: Array<UserType>,
    followingInProgress: Array<number>
};
```
- MapDispatchPropsType - типизированные thunks, принимаемые контейнерной компонентой

Например:
Принятые данные - описаны в connect

Их типизация
```
type MapDispatchPropsType = {
    followThunkCreator: (userId: number) => void,
    unfollowThunkCreator: (userId: number) => void,
    getUsersThunkCreator: (currentPage: number, pageSize: number) => void
};
```
- OwnPropsType - если есть свои данные (при их отсутствии ставим {})

- AppStateType - типизированный state из reduxStore



🚧 _Content is under development._
