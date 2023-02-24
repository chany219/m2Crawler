<template>
	<v-container>
		<v-row class="text-center">
			<v-col cols="12">
				<v-img
					:src="require('../assets/logo.svg')"
					class="my-3"
					contain
					height="200"
				/>
			</v-col>
		</v-row>
		<v-form>
			<v-container>
				<v-row>
					<v-col cols="12" md="4">
						<v-text-field
							v-model="from"
							label="PRD CD start"
							required
						></v-text-field>
					</v-col>

					<v-col cols="12" md="4">
						<v-text-field
							v-model="to"
							label="PRD CD END"
							required
						></v-text-field>
					</v-col>
					<v-col cols="12" md="4">
						<v-btn style="width: 100%" @click="main"
							>M2 CRAWLER</v-btn
						>
					</v-col>
				</v-row>
			</v-container>
		</v-form>
		<v-container>
			<v-row v-if="results.length > 0"
				><v-col class="d-flex justify-center" cols="6">
					<v-card>
						<v-card-text>상품 미변환율 </v-card-text>
						<v-card-title class="text-h2 text-lg-h2"
							>{{
								(
									(results.length /
										(Number(to) - Number(from) + 1)) *
									100
								).toFixed(2)
							}}%</v-card-title
						>
						<v-card-text
							>{{ results.length }} /
							{{ Number(to) - Number(from) + 1 }}
						</v-card-text>
					</v-card>
				</v-col>
				<v-col class="d-flex justify-center" cols="6"
					><v-card class="">
						<v-card-text>이미지 미변환율 </v-card-text>
						<v-card-title class="text-h2 text-lg-h2"
							>{{
								(
									((totalDataSrcCount + totalSrcCount) /
										(pictureCount +
											totalDataSrcCount +
											totalSrcCount)) *
									100
								).toFixed(2)
							}}
							%</v-card-title
						>
						<v-card-text
							>{{ totalDataSrcCount + totalSrcCount }} /
							{{
								pictureCount + totalDataSrcCount + totalSrcCount
							}}</v-card-text
						>
					</v-card></v-col
				>
			</v-row>
			<br />
			<v-simple-table v-if="results.length > 0">
				<tbody>
					<tr v-if="to != undefined">
						<td>
							{{ from }}
						</td>
						<td>{{ to }}</td>
					</tr>
					<tr v-if="timestampEnd != undefined">
						<td>상품 기술서 조회 시간:</td>
						<td>{{ timestampStart }} ~ {{ timestampEnd }}</td>
					</tr>
					<tr v-if="timestampEnd != undefined">
						<td>소요 시간:</td>
						<td>{{ timeSpend }} 초</td>
					</tr>
					<tr style="background-color: cornsilk">
						<td>
							{{ Number(to) - Number(from) + 1 }}개의 상품 중
							{{ results.length }}개 상품이 미변환 되었습니다
							(이미지 자체에 접근하지 못하여 (엑스박스,서버에러
							등) 변환이 이뤄지지 않은 케이스도 포함)
						</td>
						<td>
							{{ results.length }} /
							{{ Number(to) - Number(from) + 1 }} ({{
								(
									(results.length /
										(Number(to) - Number(from) + 1)) *
									100
								).toFixed(2)
							}}%)
						</td>
					</tr>
					<tr>
						<td>
							상품의 기술서 에러 (이미지 기술서 에러 ex:
							상품코드로 조회 결과 상품이 존재하지 않음)
						</td>
						<td>{{ noAccessCount }}</td>
					</tr>
					<tr>
						<td>상품 기술서 조회 API로 부터 응답을 받음</td>
						<td>{{ goodResultCount }}</td>
					</tr>
					<tr>
						<td>
							M2로부터 이미지 변환 태그를 응답 받음. (상품
							기술서가 M2의 prddesc_wrapper로 변환되어 응답됨)
						</td>
						<td>{{ wrapperCount }}</td>
					</tr>
					<tr style="background-color: cornsilk">
						<td>전체 이미지 중 미변환 비율</td>
						<td>
							{{ totalDataSrcCount + totalSrcCount }} /
							{{
								pictureCount + totalDataSrcCount + totalSrcCount
							}}
							({{
								(
									((totalDataSrcCount + totalSrcCount) /
										(pictureCount +
											totalDataSrcCount +
											totalSrcCount)) *
									100
								).toFixed(2)
							}}
							%)
						</td>
					</tr>
					<tr>
						<td>picture 태그 변환된 이미지의 수</td>
						<td>
							{{ pictureCount }}
						</td>
					</tr>
					<tr>
						<td>video 태그 변환된 이미지의 수</td>
						<td>{{ videoCount }}</td>
					</tr>
					<tr>
						<td>총 미변환 이미지의 수 (Data-Src):</td>
						<td>{{ totalDataSrcCount }}</td>
					</tr>
					<tr>
						<td>총 미변환 이미지의 수 (Src):</td>
						<td>{{ totalSrcCount }}</td>
					</tr>
					<tr>
						<td>총 미변환 GIF의 수 (Src):</td>
						<td>{{ totalGifCount }}</td>
					</tr>
				</tbody>
			</v-simple-table>
			<br />
		</v-container>

		<v-container>
			<v-data-table
				class="my-3"
				style="width: 100%"
				:headers="headers"
				:items="results"
				@click:row="(row) => handleClick(row)"
				:items-per-page="-1"
			></v-data-table>
			<!-- <template v-slot:item>
				<v-chip :color="getColor(item.count)">
					{{ item.count }}
				</v-chip>
			</template> -->
		</v-container>

		<v-overlay v-model="loading" class="align-center justify-center">
			<v-progress-circular
				color="primary"
				indeterminate
				size="64"
			></v-progress-circular>
		</v-overlay>

		<v-dialog v-model="modal" scrollable width="auto"
			><v-card>
				<v-card-title>{{ editedItem.prdCd }}</v-card-title>
				<v-divider></v-divider>
				<v-html> {{ editedItem.count }}</v-html>
				<v-html> {{ editedItem.urls }}</v-html>

				<v-divider></v-divider>
				<v-card-actions>
					<v-btn
						color="blue-darken-1"
						variant="text"
						@click="modal = false"
					>
						Close
					</v-btn>
				</v-card-actions>
			</v-card></v-dialog
		>
	</v-container>
</template>

<script>
import axios from "axios";
import cheerio from "cheerio";
axios.defaults.baseURL = "https://asm.gsshop.com/product/api/desc/";
export default {
	name: "HelloWorld",

	data: () => ({
		from: undefined,
		to: undefined,
		server: "stage",
		dataFromM2: null,
		results: [],
		step: null,
		loading: false,
		modal: false,
		timeSpend: undefined,
		totalDataSrcCount: undefined,
		totalSrcCount: undefined,
		totalGifCount: undefined,
		timestampStart: undefined,
		timestampEnd: undefined,
		editedItem: [],
		noAccessCount: 0,
		pictureCount: 0,
		goodResultCount: 0,
		wrapperCount: 0,
		exceptList: [
			"apidata.mariooutlet.com",
			":60000",
			"asset-stg.m-gs.kr",
			"https://img.fashionplus.co.kr/mall/images/goodsInsert_size02.gif",
			"https://neovision.godohosting.com/2019/WATCH/FERRAGAMO",
			"https://img.fashionplus.co.kr",
			"https://lofamscm.s3.ap-northeast-2.amazonaws.com/common/detail_title4.png?v=20190605",
		], //빈번하게 나오는 접근 불가능한 주소를 미리 입력해두면 미변환 데이터를 특정짓기더 쉬워집니다.
		headers: [
			{ text: "PRD CD", value: "prdCd" },
			{ text: "Count", value: "count" },
			{ text: "GIF", value: "gif" },
			{ text: "URL", value: "urls" },
		],
	}),
	props: {},
	computed: {
		isVaild() {
			return undefined != this.from && undefined != this.to;
		},
	},
	watch: {
		to() {
			this.results = [];
		},
		overlay(val) {
			val &&
				setTimeout(() => {
					this.overlay = false;
				}, 3000);
		},
	},
	methods: {
		create(url, options) {
			const instance = axios.create(
				Object.assign({ baseURL: url }, options)
			);
			return instance;
		},

		splitString(urls) {
			return urls.split("\n");
		},

		handleClick(row) {
			console.log(row);
			this.editedItem = row;
			this.modal = true;
		},

		main() {
			/*
			요청보낼 리스트를 만든다
			요청 Promise 리스트를 보낸다
			오는대로 하나씩 리턴 받은 결과를 파싱한다
			리스트 오는대로 result 넣어준다
			*/

			const GSSHOP_URL = "https://asm.gsshop.com/product/api/desc/";
			const gsshop = this.create(GSSHOP_URL);

			this.goodResultCount = 0;
			this.results = [];
			this.loading = true;
			this.totalSrcCount = 0;
			this.totalDataSrcCount = 0;
			this.totalGifCount = 0;
			this.wrapperCount = 0;
			this.pictureCount = 0;
			this.videoCount = 0;
			this.noAccessCount = 0;

			//요청보낼 prdCd 리스트 만들기
			const prdList = Array.from(
				{ length: this.to - this.from + 1 },
				(v, i) => i + Number(this.from)
			);

			console.log(prdList);

			//타임스탬프 요청 시작
			var startTime = new Date().getTime();
			this.timestampStart = this.timeFormater(new Date(startTime));

			Promise.all(
				prdList.map(async (prdCd) => {
					// HTTP 요청
					return await this.$axios.get(GSSHOP_URL + String(prdCd));
				})
			).then((data, prdCd) => {
				//console.log(data);
				var endTime = new Date().getTime();
				this.timestampEnd = this.timeFormater(new Date(endTime));
				this.parsing(data, prdCd);
				this.loading = false;
				this.timeSpend = (endTime - startTime) / 1000;

				console.log(
					"이미지 변환율: " +
						((this.totalDataSrcCount + this.totalSrcCount) /
							(this.pictureCount +
								this.totalDataSrcCount +
								this.totalSrcCount)) *
							(100).toFixed(2) +
						"%"
				);
				console.log(
					"미변환 이미지" +
						this.totalDataSrcCount +
						this.totalSrcCount +
						"/" +
						(this.pictureCount +
							this.totalDataSrcCount +
							this.totalSrcCount)
				);
			});
		},
		async parsing(data, prdCd) {
			data.forEach((resp, prdCd) => this.cheerio(resp, prdCd));
		},

		cheerio(resp, prdCd) {
			if (resp.data.resultCode == "0000") {
				const prdImgDescd = resp.data.prdImgDescd;
				//console.log(prdImgDescd);
				const $ = cheerio.load(prdImgDescd);
				if ($('div[id="prddesc_wrapper"]')) {
					this.wrapperCount++;
				}
				// console.log("#picture tag");
				// console.log($("picture"));
				// console.log($("picture").length);
				this.pictureCount += $("picture").length;
				$("picture").remove();
				this.videoCount += $("video").length;

				const imgList = $("img");
				if (imgList.length != 0) {
					//console.log(imgList);
					// console.log("len " + imgList.length);
					var count = 0;
					var urlList = "";
					var gifFlag = false;

					for (var index = 0; index < imgList.length; index++) {
						var skipFlag = false;
						if (undefined == imgList[index].attribs.src) {
							var imgUrldata = imgList[index].attribs["data-src"];

							for (var except of this.exceptList) {
								if (imgUrldata.indexOf(except) != -1) {
									skipFlag = true;
								}
							}

							if (skipFlag == false) {
								urlList = urlList + imgUrldata + "\n";
								this.totalDataSrcCount++;
								count++;
							}
						} else if (undefined != imgList[index].attribs.src) {
							var imgUrl = imgList[index].attribs.src;

							for (except of this.exceptList) {
								if (imgUrl.indexOf(except) != -1) {
									skipFlag = true;
								}
							}

							if (skipFlag == false) {
								urlList = urlList + imgUrl + "\n";
								this.totalSrcCount++;
								count++;
							}
						} else {
							console.log("#$$$####");
							console.log(imgList[index].attribs);
						}

						if (
							undefined != imgList[index].attribs.src &&
							imgList[index].attribs.src.indexOf(".gif") != -1
						) {
							gifFlag = true;
							this.totalGifCount++;
						} else if (
							undefined != imgList[index].attribs["data-src"] &&
							imgList[index].attribs["data-src"].indexOf(
								".gif"
							) != -1
						) {
							gifFlag = true;
							this.totalGifCount++;
						}
					}

					if (urlList != "") {
						this.results.push({
							prdCd: Number(this.from) + Number(prdCd),
							count: count,
							urls: urlList,
							gif: gifFlag == true ? "O" : "X",
						});
					}
				}
				this.goodResultCount++;
			} else {
				this.noAccessCount++;
			}
		},

		tagFilter(oriText) {
			var newText = oriText.replace(/(<([^>]+)>)/gi, "");
			return newText;
		},

		timeFormater(date) {
			var year = date.getFullYear().toString().slice(-2); //년도 뒤에 두자리
			var month = ("0" + (date.getMonth() + 1)).slice(-2); //월 2자리 (01, 02 ... 12)
			var day = ("0" + date.getDate()).slice(-2); //일 2자리 (01, 02 ... 31)
			var hour = ("0" + date.getHours()).slice(-2); //시 2자리 (00, 01 ... 23)
			var minute = ("0" + date.getMinutes()).slice(-2); //분 2자리 (00, 01 ... 59)
			var second = ("0" + date.getSeconds()).slice(-2); //초 2자리 (00, 01 ... 59)

			var returnDate =
				year +
				"." +
				month +
				"." +
				day +
				". " +
				hour +
				":" +
				minute +
				":" +
				second;

			return returnDate;
		},

		getColor(count) {
			if (count > 20) return "red";
			else if (count > 5) return "orange";
			else return "green";
		},
	},
};
</script>
<style scoped>
table,
td,
th {
	border: 1px solid gray;
	border-collapse: collapse;
}
</style>


