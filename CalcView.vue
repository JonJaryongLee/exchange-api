<template>
  <div>
    <h1>환율을 알아보자!</h1>
    <ul>
      <li v-for="exchange in exchanges" v-bind:key="exchange.id">
        <div>통화코드: {{ exchange.cur_unit }}</div>
        <div>국가/통화명: {{ exchange.cur_nm }}</div>
        <div>살 때: {{ exchange.ttb }}원</div>
        <div>팔 때: {{ exchange.tts }}원</div>
        <hr />
      </li>
    </ul>
  </div>
</template>

<script>
import axios from "axios";
// 배열 정렬 위해 lodash 설치
import _ from "lodash";
// 날짜 라이브러리 dayjs
import dayjs from "dayjs";
export default {
  data() {
    return {
      exchanges: [],
    };
  },
  created() {
    const now = dayjs();
    this.getExchangeData(now);
  },
  methods: {
    parseExchanges(exchanges) {
      const keyCountries = [
        {
          id: 1,
          cur_unit: "USD",
        },
        {
          id: 2,
          cur_unit: "EUR",
        },
        {
          id: 3,
          cur_unit: "JPY(100)",
        },
        {
          id: 4,
          cur_unit: "CNH",
        },
        {
          id: 5,
          cur_unit: "GBP",
        },
        // 원화는 항상 1이 나오므로, 아무 의미없어서 맨 마지막에 둠
        {
          id: 999,
          cur_unit: "KRW",
        },
      ];
      // 기축통화국 이외 나머지 국가는 6번부터 부여
      let cnt = 6;
      this.exchanges = exchanges.map((exchange) => {
        // 우선, 각각의 엘리먼트에 id 와 key_country 를 null 로 부여
        const format = {
          id: null,
          key_country: null,
          cur_unit: exchange.cur_unit,
          cur_nm: exchange.cur_nm,
          ttb: exchange.ttb,
          tts: exchange.tts,
        };
        // 현재 exchange 가 key_country 인지 확인하고,
        // 맞으면 길이 1의 배열로 해당 국가 뽑아내고
        // 아니라면 길이 0의 빈 배열 리턴
        const targetElement = keyCountries.filter((keyCountry) => {
          return keyCountry.cur_unit === exchange.cur_unit;
        });
        // 길이가 있다는 건 key_country 라는 뜻
        if (targetElement.length) {
          format.key_country = true;
          // 해당 주요국의 아이디 부여. 대한민국은 999
          format.id = targetElement[0].id;
          return format;
        } else {
          format.key_country = false;
          // 주요국이 아니라면 6번부터 부여
          format.id = cnt;
          cnt = cnt + 1;
          return format;
        }
      });
      // 아이디 기준 배열 정렬함
      this.exchanges = _.sortBy(this.exchanges, "id");
      // 대한민국은 맨 마지막 999번이고, 특별히 도움되는 정보가 없으므로 제외
      this.exchanges.pop();
    },
    async getExchangeData(date) {
      // 주말이나 공휴일, 평일 오전 10시 이전은 환율 정보가 빈 배열 [] 로 옴
      // 휴일은 아무리 길어도 열 번을 넘지 않기 때문에,
      // 열 번 안에 유효값이 나오면 종료
      for (let i = 0; i < 10; i++) {
        // API 사용을 위해 날짜 파싱
        const parsedDate = date.format("YYYY-MM-DD");
        // 해당 API 사용 시, 프록시서버를 두어야만 CORS 해결됨
        // 개발 과정에선 단순히 크롬 CORS 익스텐션 설치해서 테스트함
        const URL =
          "https://www.koreaexim.go.kr/site/program/financial/exchangeJSON";
        const params = {
          authkey: "kQFw9ip9VATZDdPmokPB3WDXgpP9qfwK",
          data: "AP01",
          searchdate: parsedDate,
        };
        try {
          const response = await axios.get(URL, {
            params: params,
          });
          if (response.data) {
            if (response.data.length === 0) {
              // dayjs 에서 날짜 하루 뺄 때는 subtract 사용
              // 즉, 빈 배열 [] 이면 하루씩 빼가면서 계속 시도
              date = date.subtract(1, "day");
            } else {
              // 유효한 배열을 받았을 시, 파싱함수 작동시켜 패치
              this.parseExchanges(response.data);
              break;
            }
          }
        } catch (error) {
          console.log(error);
        }
      }
    },
  },
};
</script>
