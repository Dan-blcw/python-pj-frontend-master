<script setup>
import { reactive, ref } from 'vue'
import spaFetch from "~/plugins/fetch.js";
import {useAuthStore} from "~/stores/auth.js";
</script>

<template>
    <el-dialog v-model="dialogFormVisible" title="Thông tin thanh toán" width="600">
        <div>
      <!-- T-SHIRT Size Chart -->
      <h2>T-SHIRT Size Chart</h2>
      <table>
        <thead>
          <tr>
            <th>SIZE</th>
            <th>M</th>
            <th>L</th>
            <th>XL</th>
            <th>XXL</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td>Cân Nặng (kg)</td>
            <td>&lt;60</td>
            <td>60-70</td>
            <td>70-85</td>
            <td>&gt;85</td>
          </tr>
          <tr>
            <td>Dài (cm)</td>
            <td>74</td>
            <td>77</td>
            <td>79</td>
            <td>81</td>
          </tr>
          <tr>
            <td>Ngang (cm)</td>
            <td>55</td>
            <td>57</td>
            <td>59</td>
            <td>61</td>
          </tr>
        </tbody>
      </table>
  
      <!-- SHORTS Size Chart -->
      <h2>SHORTS Size Chart</h2>
      <table>
        <thead>
          <tr>
            <th>SIZE</th>
            <th>M</th>
            <th>L</th>
            <th>XL</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td>Cân Nặng (kg)</td>
            <td>&lt;60</td>
            <td>60-75</td>
            <td>&gt;75</td>
          </tr>
          <tr>
            <td>Dài (cm)</td>
            <td>38</td>
            <td>40</td>
            <td>42</td>
          </tr>
        </tbody>
      </table>
  
      <!-- SHIRT, OUTERWEAR Size Chart -->
      <h2>SHIRT, OUTERWEAR Size Chart</h2>
      <table>
        <thead>
          <tr>
            <th>SIZE</th>
            <th>M</th>
            <th>L</th>
            <th>XL</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td>Cân Nặng (kg)</td>
            <td>&lt;60</td>
            <td>60-75</td>
            <td>&gt;75</td>
          </tr>
        </tbody>
      </table>
  
      <!-- JEANS, PANTS Size Chart -->
      <h2>JEANS, PANTS Size Chart</h2>
      <table>
        <thead>
          <tr>
            <th>SIZE</th>
            <th>M - 28</th>
            <th>L - 30</th>
            <th>XL - 32</th>
            <th>XXL - 34</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td>Cân Nặng (kg)</td>
            <td>&lt;60</td>
            <td>60-68</td>
            <td>68-75</td>
            <td>&gt;75</td>
          </tr>
        </tbody>
      </table>
    </div>
        <template #footer>
            <div class="dialog-footer">
                <el-button @click="closeModal" >Huỷ</el-button>
            </div>
        </template>
    </el-dialog>
</template>

  
  <style scoped>
  /* Add your custom styles here */
  table {
    width: 100%;
    border-collapse: collapse;
  }
  
  table, th, td {
    border: 1px solid black;
  }
  
  th, td {
    padding: 10px;
    text-align: center;
  }
  
  th {
    background-color: #f2f2f2;
  }
  </style>
  