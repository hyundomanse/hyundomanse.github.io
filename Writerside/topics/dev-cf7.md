# test
import axios from 'axios';

const getRefreshToken = mem(
async (): Promise<string |void> => {
try {
if(useAuthStore.getState().accessCode == 'INTERNAL') {
const data: {accessToken, refreshToken} = await axios.get('/api/refreshToken', {
headers: {
Authorization: `Bearer ${useAuthStore.getState().accessToken}`
}
});
return accessToken;
} else {
const data: {accessToken, refreshToken} = await axios.get('/api/refreshToken');
return accessToken;
}
} catch (error) {
authLogout();
window.location.href = '/login';
}
}, { maxAge: 2000}
);

axios.default.withCredentials = true;

axios.interceptors.request.use(
(config: InternalAxoisRequestConfig) => {
config.headers['x-language'] = useLanguageStore.getState().language;
config.headers['accept-language'] = useLanguageStore.getState().language;
const accessToken = useAuthStore.getState().accessToken;
if (accessToken) {
config.headers.Authorization = `Bearer ${accessToken}`;
}
return config;
},
(error: any) => {
return Promise.reject(error);
}
);

axois.interceptors.response.use(
async (response: any):Promise<any> => {
if(response.config.url === LOGIN_API || response.config.url === LOGIN_DEV_API) {
if (response.headers['authorization'] && response.headers['authorization'].startsWith('Bearer ')) {
const accessToken = response.headers['authorization'].replace('Bearer ', '');
useAuthStore.setState({accessToken : accessToken});
}
return response;
}
},

    async (error: any): Promise<any> => {
        const {config, response: {status}} = error;

        if (status == 500) {

        }

        if (status == 400) {
            
        }

        if (config.url === '/api/refreshToken' || status !== 401 || config.sent) {
            return Promise.reject(error);
        }

        config.sent = true;
        const accessToken = await getRefreshToken();

        if(accessToken) {
            config.headers.Authorization = `Bearer ${accessToken}`;
            return axois(config);
        }

        return Promise.reject(error);
    }



