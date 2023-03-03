<template>
	<body>
		<div
			draggable="true"
			v-for="(data, i) in copiedData"
			:key="i"
			class="categoryAndContentBox"
			v-on:dragover.prevent
			v-on:dragstart="categoryBoxDrag($event, i)"
			v-on:drop="categoryBoxDrop($event, i)"
		>
			<!-- input의 eventListener를 keydown으로작성하니 2번 호출되어서 keypress로 변경 -->
			<div
				class="contentCategory"
				v-on:click="classAddCategoryModifyModal(i, 'click')"
				v-on:mouseover="
					classAddCategoryAdd($event, i, 'hover'),
						classAddCategoryRemove($event, i, 'hover')
				"
				v-on:mouseleave="
					classRemoveRemoveButtonDragStart(),
						classRemoveCategoryAdd('hover'),
						classRemoveCategoryRemove('hover')
				"
			>
				<button class="categoryAddButton" v-on:click="categoryAdd($event)">
					+
				</button>
				<button
					class="categoryRemoveButton"
					v-on:click="categoryRemove($event, i)"
				>
					❌
				</button>
				{{ data.name }}
				<div
					class="categoryModifyModal"
					contenteditable="true"
					v-on:blur="classRemoveCategoryModifyModal('click')"
					v-on:keydown="categoryModifyInputKeyCheck($event, i)"
				></div>
			</div>
			<input
				class="contentBoxAddInput"
				placeholder="Input here..."
				v-on:keypress="contentAddInputKeyCheck($event, i)"
			/>
			<div
				v-on:dragover.prevent
				v-on:click="classAddContentModifyModal(i, j, 'click')"
				v-on:dragstart="
					classRemoveRemoveButtonDragStart(), contentBoxDrag($event, i, j)
				"
				v-on:dragenter="classAddContentBox($event, 'drag')"
				v-on:dragleave="classRemoveContentBox($event)"
				v-on:drop="contentBoxDrop($event, i, j), classRemoveContentBox($event)"
				v-on:mouseover="
					classRemoveCategoryAdd('hover'),
						classRemoveCategoryRemove('hover'),
						classAddContentRemoveButton($event, i, j, 'hover')
				"
				v-on:mouseleave="classRemoveRemoveButtonMouseLeave"
				draggable="true"
				class="contentBox"
				v-for="(data, j) in data.content"
				:key="j"
			>
				<button
					class="elementRemoveButton"
					v-on:click="contentRemove($event, i, j)"
				>
					❌
				</button>
				{{ data.text }}
				<div
					class="contentModifyModal"
					v-on:keydown="contentModifyInputKeyCheck($event, i, j)"
					v-on:click="preventEventBubbling($event)"
					v-on:blur="classRemoveModifyModal('click')"
					contenteditable="true"
				></div>
			</div>
		</div>
	</body>
</template>

<style>
	@import './assets/styles/App.css';
</style>

<script>
	class Content {
		constructor(id, newContentText) {
			this.categoryId = id;
			this.newContentText = newContentText;
		}
	}

	const backEndHost = process.env.VUE_APP_BACKENDHOST;
	const backEndPort = process.env.VUE_APP_BACKENDPORT;
	const backEndURL = backEndHost + backEndPort;

	export default {
		methods: {
			//-----------------------------CRUD관련 함수-----------------------------//
			//카테고리 추가
			categoryAdd(event) {
				if (this.copiedData.length >= 5) {
					alert('The number of maximum categories cannot exceed 5');
				} else {
					event.stopPropagation();
					fetch(`${backEndURL}/categories`, {
						method: 'POST',
						headers: {
							'Content-Type': 'application/json',
						},
					})
						.then((response) => response.json())
						.then((data) => {
							this.copiedData.push(data.data);
						});
				}
			},

			//카테고리명 수정 시 키 체크
			categoryModifyInputKeyCheck(event, i) {
				if (event.target.innerText.length < 11) {
					if (event.key === 'Escape') {
						event.target.value = '';
						event.target.blur();
					} else if (
						event.key === 'Enter' &&
						event.target.innerText.trim() === ''
					) {
						alert('Input proper text');
						event.target.blur();
					} else if (
						event.key === 'Enter' &&
						event.target.innerText.trim() !== ''
					) {
						this.copiedData[i].name = event.target.innerText;
						event.target.innerText = '';
						event.target.blur();
						//카테고리명 수정 API 호출
						fetch(`${backEndURL}/categories`, {
							method: 'PUT',
							headers: {
								'Content-Type': 'application/json',
							},
							body: JSON.stringify(this.copiedData[i]),
						})
							.then((response) => response.json())
							.catch((error) => {
								console.error('Error', error);
								alert('system error occurred');
							});
					}
				} else {
					alert('The length of input value cannot exceed 10 letters');
					event.target.blur();
				}
			},

			//카테고리 삭제
			categoryRemove(event, i) {
				event.stopPropagation();
				//카테고리 삭제 API 호출

				if (
					document.getElementsByClassName('categoryAndContentBox').length === 1
				) {
					return;
				} else {
					fetch(`${backEndURL}/categories`, {
						method: 'DELETE',
						headers: {
							'Content-Type': 'application/json',
						},
						body: JSON.stringify(this.copiedData[i]),
					})
						.then((response) => response.json())
						.catch((error) => {
							console.error('Error', error);
							alert('system error occurred');
						});
					this.copiedData.splice(i, 1);
				}
			},

			//컨텐츠 추가 시 키 체크
			contentAddInputKeyCheck(event, i) {
				if (event.target.value.length < 11) {
					if (event.key === 'Enter' && event.target.value.trim() === '') {
						event.target.value = '';
					} else if (
						event.key === 'Enter' &&
						event.target.value.trim() !== '' &&
						this.copiedData[i].content.length >= 6
					) {
						alert(
							'The number of maximum contents per category cannot exceed 6'
						);
						event.target.value = '';
						event.target.blur();
					} else if (
						event.key === 'Enter' &&
						event.target.value.trim() !== '' &&
						this.copiedData[i].content.length < 6
					) {
						const newContent = new Content(
							this.copiedData[i].id,
							event.target.value.trim()
						);
						//컨텐츠 추가 API 호출
						fetch(`${backEndURL}/contents`, {
							method: 'POST',
							headers: {
								'Content-Type': 'application/json',
							},
							body: JSON.stringify(newContent),
						})
							.then((response) => response.json())
							.then((data) => this.copiedData[i].content.push(data.data))
							.catch((error) => {
								console.error('Error', error);
								alert('system error occurred');
							});
						event.target.value = '';
					}
				} else {
					alert('The length of input value cannot exceed 10 letters');
					event.target.value = '';
					event.target.blur();
				}
			},

			//컨텐츠 수정 시 키 체크
			contentModifyInputKeyCheck: async function (event, i, j) {
				if (event.target.innerText.length < 11) {
					if (event.key === 'Escape') {
						event.target.value = '';
						event.target.blur();
					} else if (
						event.key === 'Enter' &&
						event.target.innerText.trim() === ''
					) {
						alert('Input proper text');
						event.target.blur();
					} else if (
						event.key === 'Enter' &&
						event.target.innerText.trim() !== ''
					) {
						this.copiedData[i].content[j].text = event.target.innerText;
						event.target.blur();
						//컨텐츠 수정 API 호출
						fetch(`${backEndURL}/contents`, {
							method: 'PUT',
							headers: {
								'Content-Type': 'application/json',
							},
							body: JSON.stringify(this.copiedData[i].content[j]),
						})
							.then((response) => response.json())
							.catch((error) => {
								console.error('Error', error);
								alert('system error occurred');
							});
					}
				} else {
					alert('The length of input value cannot exceed 10 letters');
					event.target.blur();
				}
			},

			//컨텐츠 삭제
			contentRemove(event, i, j) {
				event.stopPropagation();
				//컨텐츠 삭제 API 호출
				fetch(`${backEndURL}/contents`, {
					method: 'DELETE',
					headers: {
						'Content-Type': 'application/json',
					},
					body: JSON.stringify(this.copiedData[i].content[j]),
				})
					.then((response) => response.json())
					.catch((error) => {
						console.error('Error', error);
					});
				this.copiedData[i].content.splice(j, 1);
			},

			//-----------------------------Drag&Drop관련 함수-----------------------------//
			//카테고리박스 드래그
			categoryBoxDrag(event, i) {
				event.stopPropagation();
				event.dataTransfer.dropEffect = 'move';
				event.dataTransfer.effectAllowed = 'move';
				this.contentToSplice = { ...this.copiedData[i] };
				this.contentToSpliceCoordinate = [i, 'None'];
			},
			//카테고리박스 드랍
			categoryBoxDrop(event, i) {
				event.stopPropagation();
				if (i === this.contentToSpliceCoordinate[0]) {
					return;
				} else {
					fetch(`${backEndURL}/orders/categories`, {
						method: 'PUT',
						headers: {
							'Content-Type': 'application/json',
						},
						body: JSON.stringify({
							sourceOrderInf: this.contentToSpliceCoordinate[0],
							targetOrderInf: i,
						}),
					});

					if (i > this.contentToSpliceCoordinate[0]) {
						this.copiedData.splice(i, 0, this.contentToSplice);
						this.copiedData.splice(
							this.contentToSpliceCoordinate[0],
							0,
							this.copiedData[i + 1]
						);
						this.copiedData.splice(i + 2, 1);
						this.copiedData.splice(this.contentToSpliceCoordinate[0] + 1, 1);
						this.contentToSplice = {};
						this.contentToSpliceCoordinate = [];
					} else if (i < this.contentToSpliceCoordinate[0]) {
						this.copiedData.splice(i, 0, this.contentToSplice);
						this.copiedData.splice(
							this.contentToSpliceCoordinate[0] + 2,
							0,
							this.copiedData[i + 1]
						);
						this.copiedData.splice(this.contentToSpliceCoordinate[0] + 1, 1);
						this.copiedData.splice(i + 1, 1);
					} else {
						return;
					}
				}
			},

			//컨텐츠박스 드래그
			contentBoxDrag(event, i, j) {
				event.stopPropagation();
				event.dataTransfer.dropEffect = 'move';
				event.dataTransfer.effectAllowed = 'move';
				this.contentToSplice = { ...this.copiedData[i].content[j] };
				this.contentToSpliceCoordinate = [i, j];
			},

			//컨텐츠박스 드랍
			contentBoxDrop(event, i, j) {
				event.stopPropagation();
				if (this.contentToSpliceCoordinate[0] === i) {
					fetch(`${backEndURL}/orders/contents`, {
						method: 'PUT',
						headers: {
							'Content-Type': 'application/json',
						},
						body: JSON.stringify({
							sourceContentOrderInf: this.contentToSplice.orderInf,
							targetContentOrderInf: this.copiedData[i].content[j].orderInf,
						}),
					});
					if (j > this.contentToSpliceCoordinate[1]) {
						this.copiedData[i].content.splice(j, 0, this.contentToSplice);
						this.copiedData[i].content.splice(
							this.contentToSpliceCoordinate[1],
							0,
							this.copiedData[i].content[j + 1]
						);
						this.copiedData[i].content.splice(j + 2, 1);
						this.copiedData[i].content.splice(
							this.contentToSpliceCoordinate[1] + 1,
							1
						);
						this.contentToSplice = {};
						this.contentToSpliceCoordinate = [];
					} else if (j < this.contentToSpliceCoordinate[1]) {
						this.copiedData[i].content.splice(j, 0, this.contentToSplice);
						this.copiedData[i].content.splice(
							this.contentToSpliceCoordinate[1] + 2,
							0,
							this.copiedData[i].content[j + 1]
						);
						this.copiedData[i].content.splice(
							this.contentToSpliceCoordinate[1] + 1,
							1
						);
						this.copiedData[i].content.splice(j + 1, 1);
					} else {
						return;
					}
				} else {
					if (this.copiedData[i].content.length < 6) {
						fetch(`${backEndURL}/orders/contents`, {
							method: 'PATCH',
							headers: {
								'Content-Type': 'application/json',
							},
							body: JSON.stringify({
								sourceContentId:
									this.copiedData[this.contentToSpliceCoordinate[0]].content[
										this.contentToSpliceCoordinate[1]
									].id,
								sourceCategoryId:
									this.copiedData[this.contentToSpliceCoordinate[0]].id,
								targetCategoryId: this.copiedData[i].id,
								sourceContentOrderInf:
									this.copiedData[this.contentToSpliceCoordinate[0]].content[
										this.contentToSpliceCoordinate[1]
									].orderInf,
								targetContentOrderInf: this.copiedData[i].content[j].orderInf,
							}),
						});
						this.copiedData[i].content.splice(j, 0, this.contentToSplice);
						this.copiedData[this.contentToSpliceCoordinate[0]].content.splice(
							this.contentToSpliceCoordinate[1],
							1
						);
						this.contentToSplice = {};
						this.contentToSpliceCoordinate = [];
					} else {
						alert(
							'The number of maximum contents per category cannot exceed 6'
						);
						return;
					}
				}
			},

			// 컨텐츠 삭제 버튼 클릭 시 아래에 비치는 창이 클릭 되는 현상 방지
			preventEventBubbling(event) {
				event.stopPropagation();
			},

			//-----------------------------CSS관련 함수-----------------------------//
			//카테고리박스 클릭 시 수정을 위한 모달창 클래스 추가
			classAddCategoryModifyModal(i, csskeyWord) {
				/* eslint-disable */
				if (
					document.getElementsByClassName(`categoryModifyModal ${csskeyWord}`)
						.length > 0
				) {
					document
						.getElementsByClassName(`categoryModifyModal ${csskeyWord}`)[0]
						.classList.remove(csskeyWord);
					document
						.getElementsByClassName('categoryModifyModal')
						[i].classList.add(csskeyWord);
					document
						.getElementsByClassName(`categoryModifyModal ${csskeyWord}`)[0]
						.focus();
				} else {
					document
						.getElementsByClassName('categoryModifyModal')
						[i].classList.add(csskeyWord);
					document;
					document
						.getElementsByClassName(`categoryModifyModal ${csskeyWord}`)[0]
						.focus();
				}
				/* eslint-enable */
			},
			//카테고리박스 blur 시 클래스 삭제
			classRemoveCategoryAdd(csskeyWord) {
				/* eslint-disable */
				document
					.getElementsByClassName(`categoryAddButton ${csskeyWord}`)[0]
					?.classList.remove(csskeyWord);
				/* eslint-enable */
			},
			//카테고리박스 hover 시 카테고리추가버튼 클래스 추가
			classAddCategoryAdd(event, i, csskeyWord) {
				/* eslint-disable */
				event.stopPropagation();
				document
					.getElementsByClassName('categoryAddButton')
					[i].classList.add(csskeyWord);
				/* eslint-enable */
			},
			//카테고리박스 hover 시 카테고리삭제버튼 클래스 추가
			classAddCategoryRemove(event, i, csskeyWord) {
				/* eslint-disable */
				event.stopPropagation();
				document
					.getElementsByClassName('categoryRemoveButton')
					[i].classList.add(csskeyWord);
				/* eslint-enable */
			},
			//카테고리박스 hover 끝날 시 카테고리삭제버튼 클래스 삭제
			classRemoveCategoryRemove(csskeyWord) {
				/* eslint-disable */
				document
					.getElementsByClassName(`categoryRemoveButton ${csskeyWord}`)[0]
					?.classList.remove(csskeyWord);
				/* eslint-enable */
			},
			//카테고리 수정 입력 창 blur 시 클래스 삭제
			classRemoveCategoryModifyModal(csskeyWord) {
				/* eslint-disable */
				document
					.getElementsByClassName(`categoryModifyModal ${csskeyWord}`)[0]
					.classList.remove(csskeyWord);
				/* eslint-enable */
			},
			//컨텐츠박스 mouse-leave시 컨텐츠삭제버튼 클래스 삭제
			classAddContentRemoveButton(event, i, j, csskeyWord) {
				/* eslint-disable */
				event.stopPropagation();
				if (i === 0) {
					document
						.getElementsByClassName('elementRemoveButton')
						[j].classList.add(csskeyWord);
				} else {
					for (
						let categoriesCount = 0;
						categoriesCount < i;
						categoriesCount++
					) {
						this.contentCount +=
							this.copiedData[categoriesCount].content.length;
					}
					document
						.getElementsByClassName('elementRemoveButton')
						[this.contentCount + j].classList.add(csskeyWord);
					this.contentCount = 0;
				}
				/* eslint-enable */
			},
			//컨텐츠수정 입력 창 blur 시 클래스 삭제
			classRemoveModifyModal(csskeyWord) {
				document.getElementsByClassName(
					'contentModifyModal click'
				)[0].innerText = '';
				document
					.getElementsByClassName(`contentModifyModal ${csskeyWord}`)[0]
					.classList.remove(csskeyWord);
			},
			//컨텐츠박스 blur 시 수정을 위한 모달창 클래스 추가
			classAddContentModifyModal(i, j, csskeyWord) {
				/* eslint-disable */
				if (i === 0) {
					document
						.getElementsByClassName('contentModifyModal')
						[j].classList.add(csskeyWord);
					document
						.getElementsByClassName(`contentModifyModal ${csskeyWord}`)[0]
						.focus();
				} else {
					for (
						let categoriesCount = 0;
						categoriesCount < i;
						categoriesCount++
					) {
						this.contentCount +=
							this.copiedData[categoriesCount].content.length;
					}
					document
						.getElementsByClassName('contentModifyModal')
						[this.contentCount + j].classList.add(csskeyWord);
					document
						.getElementsByClassName(`contentModifyModal ${csskeyWord}`)[0]
						.focus();
					this.contentCount = 0;
					/* eslint-enable */
				}
			},
			//컨텐츠박스 drag-in시 클래스 추가
			classAddContentBox(event, csskeyWord) {
				event.target.classList.add(csskeyWord);
			},
			//컨텐츠박스 drag-leave시 클래스 삭제
			classRemoveContentBox(event) {
				if (event.target.classList.value === 'contentBox drag') {
					event.target.classList.remove('drag');
				}
			},
			//컨텐츠박스 drag-start시 컨텐츠삭제버튼 클래스 삭제
			classRemoveRemoveButtonDragStart() {
				document
					.getElementsByClassName('elementRemoveButton hover')[0]
					?.classList.remove('hover');
			},
			//컨텐츠박스 mouse-leave시 컨텐츠삭제버튼 클래스 삭제
			classRemoveRemoveButtonMouseLeave() {
				document
					.getElementsByClassName('elementRemoveButton hover')[0]
					?.classList.remove('hover');
				/* eslint-enable */
			},
		},

		data() {
			return {
				copiedData: [],
				contentToSplice: {},
				contentToSpliceCoordinate: [],
				contentCount: 0,
			};
		},

		beforeCreate() {
			fetch(`${backEndURL}/categories/all`)
				.then((response) => response.json())
				.then((data) => (this.copiedData = [...data.data]))
				.catch((error) => {
					console.error(error);
					alert('system initializing failed');
				});
		},
	};
</script>
