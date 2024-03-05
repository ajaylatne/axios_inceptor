import axios from "axios";
const axios_instance = axios.create({
  baseURL: 'http://localhost:8000',
});


function fetchAllData(){
    const config = {method: 'GET'}
    return axios_instance(config)
}

async function login_cred(data){
    try {
        const response = await axios_instance.post('token/', data);
        const { refresh, access } = response.data;
        localStorage.setItem('access', access);
        localStorage.setItem('refresh', refresh);

      } catch (error) {
        console.log("plz login")
      }
}

function logout(){
  localStorage.removeItem('access');
  localStorage.removeItem('refresh');
}







/**********************************************************/

let count = 0;
async function getAccessToken(){
  const refresh = localStorage.getItem('refresh');
  return axios_instance.post('token/refresh/', { refresh }).then().catch(()=>{count=0; logout()})
}


axios_instance.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('access');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);




axios_instance.interceptors.response.use(
  (response) => response,
  async (error) => {
    const originalRequest = error.config;
    
    if (error.response.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;
      count += 1;
      if (count>2){return}
      try {
        const response = await getAccessToken()
        console.log(response)
        const { access } = response.data;

        localStorage.setItem('access', access);
     
        originalRequest.headers.Authorization = `Bearer ${access}`;

        return axios_instance(originalRequest);
      } catch (error) { 
 
        return Promise.reject(error);
      }
    }

    return Promise.reject(error);
  }
);

export { fetchAllData, login_cred, logout }
