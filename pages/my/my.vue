<template>
	<view class="page">
		<view class="profile-container">
			<!-- 个人信息区域 -->
			<view class="profile-header">
				<view class="avatar-container">
					<image class="avatar" :src="userInfo.avatar" mode="aspectFill" />
					<view class="edit-avatar cursor-pointer" @click="handleEditAvatar">
						<uni-icons type="camera-filled" size="20" color="#FFFFFF" />
					</view>
				</view>
				<view class="name-container">
					<text class="name">{{ userInfo.username }}</text>
					<uni-icons type="compose" size="16" color="#666666" class="edit-icon cursor-pointer"
						@click="handleEditName" />
				</view>
				<view class="intro-container">
					<text class="intro">{{ userInfo.intro }}</text>
					<uni-icons type="compose" size="16" color="#666666" class="edit-icon cursor-pointer"
						@click="handleEditIntro" />
				</view>
			</view>

			<!-- 基本信息区域 -->
			<view class="info-section">
				<view class="info-item">
					<text class="info-label">邮箱</text>
					<view class="info-value">
						<text>{{ userInfo.email || '未设置' }}</text>
						<uni-icons v-if="validatePhone" type="checkmarkempty" size="16" color="#4CD964" />
						<uni-icons type="compose" size="16" color="#999" @click="showEmailEditDialog" />
					</view>
				</view>
				<view class="info-item">
					<text class="info-label">手机号</text>
					<view class="info-value">
						<text>{{ userInfo.phone || '未设置' }}</text>
						<uni-icons v-if="validateEmail" type="checkmarkempty" size="16" color="#4CD964" />
						<uni-icons type="compose" size="16" color="#999" @click="showPhoneEditDialog" />
					</view>
				</view>
			</view>

			<!-- 功能列表区域 -->
			<view class="function-list">
				<view class="function-item cursor-pointer" @click="handleSecurity">
					<view class="item-left">
						<uni-icons type="locked" size="20" color="#36B4FE" />
						<text class="item-text">账户安全</text>
					</view>
					<uni-icons type="right" size="16" color="#CCCCCC" />
				</view>
				<view class="function-item cursor-pointer" @click="handlePassword">
					<view class="item-left">
						<image src="../../static/my/密码修改.png" mode="aspectFit"></image>
						<text class="item-text">密码修改</text>
					</view>
					<uni-icons type="right" size="16" color="#CCCCCC" />
				</view>
				<view class="function-item cursor-pointer" @click="handleNotification">
					<view class="item-left">
						<uni-icons type="notification" size="20" color="#9C7AFE" />
						<text class="item-text">通知设置</text>
					</view>
					<uni-icons type="right" size="16" color="#CCCCCC" />
				</view>
				<view class="function-item cursor-pointer" @click="handlePrivacy">
					<view class="item-left">
						<image src="../../static/my/隐私设置.png" mode="aspectFit"></image>
						<text class="item-text">隐私设置</text>
					</view>
					<uni-icons type="right" size="16" color="#CCCCCC" />
				</view>
				<view class="function-item cursor-pointer" @click="handleHelp">
					<view class="item-left">
						<uni-icons type="help" size="20" color="#8A8A8A" />
						<text class="item-text">帮助中心</text>
					</view>
					<uni-icons type="right" size="16" color="#CCCCCC" />
				</view>
				<view class="function-item cursor-pointer" @click="handleAbout">
					<view class="item-left">
						<uni-icons type="info" size="20" color="#8A8A8A" />
						<text class="item-text">关于我们</text>
					</view>
					<uni-icons type="right" size="16" color="#CCCCCC" />
				</view>
			</view>

			<!-- 退出登录按钮 -->
			<button class="logout-btn cursor-pointer" @click="handleLogout">退出登录</button>
		</view>

		<!-- 编辑弹窗 -->
		<uni-popup ref="editPopup" type="dialog" @maskClick="handleDialogClose">
			<uni-popup-dialog mode="input" :title="dialogTitle" :value="dialogValue" :placeholder="dialogPlaceholder"
				@confirm="handleDialogConfirm" @close="handleDialogClose" />
		</uni-popup>

		<!-- 密码修改弹窗 -->
		<uni-popup ref="passwordPopup" type="dialog">
			<view class="password-form">
				<text class="dialog-title">修改密码</text>

				<!-- 旧密码输入 -->
				<view class="form-item">
					<text class="item-label">旧密码</text>
					<input v-model="passwordDialog.oldPassword" type="password" placeholder="请输入旧密码"
						class="input-field" />
				</view>

				<!-- 新密码输入 -->
				<view class="form-item">
					<text class="item-label">新密码</text>
					<input v-model="passwordDialog.newPassword" type="password" placeholder="请输入新密码(至少6位)"
						class="input-field" />
				</view>

				<!-- 确认新密码 -->
				<view class="form-item">
					<text class="item-label">确认新密码</text>
					<input v-model="passwordDialog.confirmPassword" type="password" placeholder="请再次输入新密码"
						class="input-field" />
				</view>

				<view class="button-group">
					<button @click="closePasswordDialog" class="cancel-btn">取消</button>
					<button @click="handlePasswordConfirm" class="confirm-btn">确定</button>
				</view>
			</view>
		</uni-popup>

		<uni-popup ref="emailEditDialog" type="dialog">
			<view class="edit-dialog">
				<text class="dialog-title">修改邮箱</text>
				<input v-model="emailEditValue" type="text" placeholder="请输入新邮箱" class="dialog-input" />
				<view class="dialog-buttons">
					<button @click="cancelEmailEdit" class="dialog-button cancel">取消</button>
					<button @click="handleEmailConfirm" class="dialog-button confirm">确定</button>
				</view>
			</view>
		</uni-popup>

		<!-- 手机号编辑弹窗 -->
		<uni-popup ref="phoneEditDialog" type="dialog">
			<view class="edit-dialog">
				<text class="dialog-title">修改手机号</text>
				<input v-model="phoneEditValue" type="number" placeholder="请输入新手机号" class="dialog-input"
					maxlength="11" />

				<view class="verify-code-section" v-if="phoneEditing">
					<input v-model="verifyCode" type="number" placeholder="请输入验证码" class="verify-input" maxlength="6" />
					<button class="verify-btn" :disabled="countdown > 0" @click="sendVerifyCode">
						{{ countdown > 0 ? `${countdown}秒后重试` : '获取验证码' }}
					</button>
				</view>

				<view class="dialog-buttons">
					<button @click="cancelPhoneEdit" class="dialog-button cancel">取消</button>
					<button @click="handlePhoneConfirm" class="dialog-button confirm">确定</button>
				</view>
			</view>
		</uni-popup>

	</view>
</template>

<script lang="ts">
	import axios from 'axios';
	import { defineComponent } from 'vue';

	export default defineComponent({
		data() {
			return {
				dialogTitle: '',
				dialogContent: '',
				dialogValue: '',
				dialogPlaceholder: '',
				currentEditType: '',
				userInfo: {
					avatar: "../../static/默认头像.png",
					username: '李小明',
					intro: '热爱生活，专注时间管理，让每一天都过得更有意义。',
					email: '暂未绑定',
					phone: '暂未绑定'
				},
				passwordDialog: {
					oldPassword: '',
					newPassword: '',
					confirmPassword: '',
				},

				// 邮箱编辑相关
				emailEditValue: '',
				emailEditing: false,

				// 手机号编辑相关
				phoneEditValue: '',
				phoneEditing: false,
				verifyCode: '',
				countdown: 0,
				currentPhone: ''
			}
		},

		methods: {
			handleEditAvatar() {
				uni.showToast({
					title: '编辑头像功能开发中',
					icon: 'none'
				});
			},
			handleEditName() {
				this.dialogTitle = '修改昵称';
				this.dialogPlaceholder = '请输入新的昵称';
				this.dialogValue = this.userInfo.username;
				this.currentEditType = 'name';
				this.$refs.editPopup.open();
			},

			// 编辑简介
			handleEditIntro() {
				this.dialogTitle = '修改简介';
				this.dialogPlaceholder = '请输入新的简介';
				this.dialogValue = this.userInfo.intro;
				this.currentEditType = 'intro';
				this.$refs.editPopup.open();
			},

			// 确认修改
			async handleDialogConfirm(value) {
				try {
					if (!value || value.trim() === '') {
						uni.showToast({
							title: '输入不能为空',
							icon: 'none'
						});
						return;
					}

					if (this.currentEditType === 'name') {
						this.userInfo.username = value;
					} else if (this.currentEditType === 'intro') {
						this.userInfo.intro = value;
					}

					// 发起API请求保存修改
					const token = uni.getStorageSync('token');
					if (token) {
						const response = await axios({
							url: '/user/update',
							method: 'POST',
							headers: { 'satoken': token },
							data: this.userInfo
						});

						if (response.data.code === 200) {
							uni.showToast({ title: '保存成功', icon: 'success' });
							uni.setStorageSync('user', this.userInfo);
						}
					}
				} catch (error) {
					console.error('保存失败:', error);
					uni.showToast({
						title: '保存失败',
						icon: 'none'
					});
				} finally {
					this.$refs.editPopup.close();
				}
			},
			handleDialogClose() {
				this.$refs.editPopup.close();
			},
			handleSecurity() {
				uni.showToast({
					title: '账户安全功能开发中',
					icon: 'none'
				});
			},
			handlePassword() {
				this.passwordDialog = {
					oldPassword: '',
					newPassword: '',
					confirmPassword: ''
				};
				this.$refs.passwordPopup.open();
			},
			// 密码弹窗确认
			async handlePasswordConfirm() {
				const { oldPassword, newPassword, confirmPassword } = this.passwordDialog;

				// 前端基本验证
				if (!oldPassword) {
					uni.showToast({ title: '请输入旧密码', icon: 'none' });
					return;
				}

				if (!newPassword || newPassword.length < 6) {
					uni.showToast({ title: '新密码至少6位', icon: 'none' });
					return;
				}

				if (newPassword !== confirmPassword) {
					uni.showToast({ title: '两次输入密码不一致', icon: 'none' });
					return;
				}

				// 调用提交方法
				await this.submitPasswordChange(oldPassword, newPassword);
			},
			// 提交密码修改
			async submitPasswordChange(oldPassword, newPassword) {
				const token = uni.getStorageSync('token');
				if (!token) {
					uni.showToast({ title: '请先登录', icon: 'none' });
					return;
				}

				uni.showLoading({ title: '修改中...', mask: true });

				try {
					const requestData = {
						...this.userInfo,    // 用户基本信息
						oldPassword,        // 旧密码
						password: newPassword // 新密码
					};

					const response = await axios({
						url: '/user/updatePassword',
						method: 'POST',
						headers: {
							'satoken': token,
							'Content-Type': 'application/json'
						},
						data: requestData
					});

					if (response.data.code === 200) {
						uni.showToast({ title: '密码修改成功', icon: 'success' });

						// 关闭弹窗
						this.$refs.passwordPopup.close();

						// 密码修改成功后可选退出登录
						setTimeout(() => {
							this.handleLogout();
						}, 1000);
					} else {
						// 显示后端返回的错误信息
						uni.showToast({
							title: response.data.msg || '密码修改失败',
							icon: 'none'
						});
					}
				} catch (error) {
					console.error('密码修改失败:', error);
					uni.showToast({
						title: error.response?.data?.msg || '修改失败，请重试',
						icon: 'none'
					});
				} finally {
					uni.hideLoading();
				}
			},
			// 关闭密码弹窗
			closePasswordDialog() {
				this.$refs.passwordPopup.close();
			},
			handleNotification() {
				uni.showToast({
					title: '通知设置功能开发中',
					icon: 'none'
				});
			},
			handlePrivacy() {
				uni.showToast({
					title: '隐私设置功能开发中',
					icon: 'none'
				});
			},
			handleHelp() {
				uni.showToast({
					title: '帮助中心功能开发中',
					icon: 'none'
				});
			},
			handleAbout() {
				uni.showToast({
					title: '关于我们功能开发中',
					icon: 'none'
				});
			},
			handleLogout() {
				uni.showModal({
					title: '提示',
					content: '确定要退出登录吗？',
					success: (res) => {
						if (res.confirm) {
							// 获取本地存储的 token
							const token = uni.getStorageSync('token');
							if (!token) {
								uni.showToast({
									title: '未找到登录信息',
									icon: 'none'
								});
								return;
							}

							// 发送退出登录请求，将 token 添加到 header 中
							axios({
								url: '/user/logout',
								method: 'POST',
								headers: {
									'satoken': token
								}
							})
								.then(() => {
									uni.showToast({
										title: '已退出登录',
										icon: 'success'
									});
									// 清除本地存储的 token并跳转至login页
									uni.removeStorageSync('token');
									uni.removeStorageSync('user');
									uni.navigateTo({
										url: '/pages/login/login'
									})
								})
								.catch((err) => {
									console.error('退出登录失败:', err);
									uni.showToast({
										title: '退出登录失败',
										icon: 'none'
									});
								});
						}
					}
				});
			},

			// 显示邮箱编辑弹窗
			showEmailEditDialog() {
				this.emailEditValue = this.userInfo.email || '';
				this.emailEditing = true;
				this.$refs.emailEditDialog.open();
			},

			// 邮箱输入处理
			handleEmailInput(value) {
				this.emailEditValue = value;
			},

			// 确认邮箱修改
			async handleEmailConfirm() {
				if (!this.validateEmail(this.emailEditValue)) {
					uni.showToast({ title: '请输入正确的邮箱格式', icon: 'none' });
					return;
				}

				try {
					await this.updateUserInfo({ email: this.emailEditValue });
					this.$refs.emailEditDialog.close();
				} catch (error) {
					console.error('邮箱修改失败:', error);
				}
			},

			// 取消邮箱编辑
			cancelEmailEdit() {
				this.$refs.emailEditDialog.close();
				this.emailEditing = false;
				this.emailEditValue = '';
			},

			// 显示手机号编辑弹窗
			showPhoneEditDialog() {
				this.phoneEditValue = this.userInfo.phone || '';
				this.phoneEditing = false;
				this.verifyCode = '';
				this.$refs.phoneEditDialog.open();
			},

			// 手机号输入处理
			handlePhoneInput(value) {
				this.phoneEditValue = value;
			},

			// 确认手机号修改
			async handlePhoneConfirm() {
				// 第一阶段：验证手机号格式
				if (!this.phoneEditing) {
					if (!this.validatePhone(this.phoneEditValue)) {
						uni.showToast({ title: '请输入正确的手机号', icon: 'none' });
						return;
					}

					this.currentPhone = this.phoneEditValue;
					this.phoneEditing = true;
					return;
				}

				// 第二阶段：验证验证码
				if (!this.verifyCode) {
					uni.showToast({ title: '请输入验证码', icon: 'none' });
					return;
				}

				try {
					// 验证验证码
					const verifyRes = await axios.post('/user/verifyPhone', {
						phone: this.currentPhone,
						code: this.verifyCode
					});

					if (verifyRes.data.code !== 200) {
						const errorMsg = verifyRes.data.msg;
						uni.showToast({
							title: errorMsg || '验证码错误',
							icon: 'none'
						});
						return;
						// throw new Error(verifyRes.data.msg || '验证码错误');
					}else{
						const dataMsg = verifyRes.data.msg;
						uni.showToast({
							title:dataMsg,
							icon:'none'
						})
					}

					// 更新手机号
					await this.updateUserInfo({ phone: this.currentPhone });

					// 关闭弹窗
					this.$refs.phoneEditDialog.close();
				} catch (error) {
					let errorMessage = '手机号更新失败';

					// 优先使用服务器返回的错误信息
					if (error.response && error.response.data && error.response.data.msg) {
						errorMessage = error.response.data.msg;
					} else if (error.message) {
						// 其次使用错误对象的message
						errorMessage = error.message;
					}

					uni.showToast({
						title: errorMessage,
						icon: 'none',
						duration: 2000
					});
				}
			},

			// 取消手机号编辑
			// 取消手机号编辑
			cancelPhoneEdit() {
				this.$refs.phoneEditDialog.close();
				this.phoneEditing = false;
				this.phoneEditValue = '';
				this.verifyCode = '';
				this.countdown = 0;
			},

			// 发送验证码
			async sendVerifyCode() {
				if (this.countdown > 0) return;

				try {
					const res = await axios.post('/user/sms/send', {
						phone: this.currentPhone
					});

					if (res.data.code === 200) {
						uni.showToast({ title: '验证码已发送', icon: 'none' });
						this.startCountdown();
					} else {
						throw new Error(res.data.msg || '发送失败');
					}
				} catch (error) {
					uni.showToast({
						title: error.message || '验证码发送失败',
						icon: 'none'
					});
				}
			},

			// 开始倒计时
			startCountdown() {
				this.countdown = 60;
				const timer = setInterval(() => {
					this.countdown--;
					if (this.countdown <= 0) {
						clearInterval(timer);
					}
				}, 1000);
			},

			// 更新用户信息
			async updateUserInfo(data) {
				const token = uni.getStorageSync('token');
				if (!token) return;

				uni.showLoading({ title: '保存中...', mask: true });

				try {
					const response = await axios({
						url: '/user/update',
						method: 'POST',
						headers: { 'satoken': token },
						data: {
							...this.userInfo,
							...data
						}
					});

					if (response.data.code === 200) {
						this.userInfo = { ...this.userInfo, ...data };
						uni.showToast({ title: '修改成功', icon: 'success' });
					} else {
						throw new Error(response.data.msg || '修改失败');
					}
				} catch (error) {
					uni.showToast({
						title: error.message || '修改失败',
						icon: 'none'
					});
					throw error;
				} finally {
					uni.hideLoading();
				}
			},

			// 验证邮箱格式
			validateEmail(email) {
				return /^\w+([-+.]\w+)*@\w+([-.]\w+)*\.\w+([-.]\w+)*$/.test(email);
			},

			// 验证手机号格式
			validatePhone(phone) {
				return /^1[3-9]\d{9}$/.test(phone);
			}




		},
		created() {
			this.userInfo = uni.getStorageSync('user')
		}
	});
</script>


<style>
	page {
		height: 100%;
		background-color: #F8F8F8;
	}

	.profile-container {
		padding: 40rpx 30rpx;
	}

	.profile-header {
		display: flex;
		flex-direction: column;
		align-items: center;
		padding: 40rpx;
		background-color: #FFFFFF;
		border-radius: 16rpx;
		margin-bottom: 20rpx;
	}

	.avatar-container {
		position: relative;
		margin-bottom: 20rpx;
	}

	.avatar {
		width: 160rpx;
		height: 160rpx;
		border-radius: 80rpx;
	}

	.edit-avatar {
		position: absolute;
		right: 0;
		bottom: 0;
		width: 40rpx;
		height: 40rpx;
		background-color: #36B4FE;
		border-radius: 20rpx;
		display: flex;
		justify-content: center;
		align-items: center;
	}

	.name-container {
		display: flex;
		align-items: center;
		margin-bottom: 16rpx;
	}

	.name {
		font-size: 18px;
		font-weight: 500;
		color: #333333;
		margin-right: 12rpx;
	}

	.intro-container {
		display: flex;
		align-items: center;
	}

	.intro {
		font-size: 14px;
		color: #666666;
		margin-right: 12rpx;
	}

	.edit-icon {
		padding: 8rpx;
	}

	.info-section {
		background-color: #FFFFFF;
		border-radius: 16rpx;
		padding: 30rpx;
		margin-bottom: 20rpx;
	}

	.info-item {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 20rpx 0;
	}

	.info-item:first-child {
		padding-top: 0;
	}

	.info-item:last-child {
		padding-bottom: 0;
	}

	.info-label {
		font-size: 14px;
		color: #999999;
	}

	.info-value {
		font-size: 14px;
		color: #333333;
		display: flex;
		align-items: center;
	}

	.info-value text {
		margin-right: 8rpx;
	}

	.function-list {
		background-color: #FFFFFF;
		border-radius: 16rpx;
		padding: 0 30rpx;
	}

	.function-item {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 30rpx 0;
		border-bottom: 1px solid #F5F5F5;
	}

	.function-item:last-child {
		border-bottom: none;
	}

	.item-left {
		display: flex;
		align-items: center;
	}

	.item-left image {
		width: 20px;
		height: 20px;
	}

	.item-text {
		font-size: 14px;
		color: #333333;
		margin-left: 16rpx;
	}

	.logout-btn {
		margin-top: 60rpx;
		width: 100%;
		height: 88rpx;
		line-height: 88rpx;
		background-color: #FEE7E7;
		color: #FF5151;
		font-size: 16px;
		border-radius: 44rpx;
		border: none;
	}

	.logout-btn::after {
		border: none;
	}

	.cursor-pointer {
		cursor: pointer;
	}

	.password-form {
		padding: 20px;
		background: #fff;
		border-radius: 12px;
		width: 80vw;
	}

	.dialog-title {
		font-size: 18px;
		font-weight: bold;
		display: block;
		text-align: center;
		margin-bottom: 20px;
	}

	.form-item {
		margin-bottom: 16px;
	}

	.item-label {
		display: block;
		margin-bottom: 6px;
		font-size: 14px;
		color: #666;
	}

	.input-field {
		border: 1px solid #ddd;
		padding: 10px;
		border-radius: 6px;
		width: 100%;
		/* box-sizing: border-box; */
	}

	.button-group {
		display: flex;
		justify-content: space-between;
		margin-top: 20px;
	}

	.cancel-btn,
	.confirm-btn {
		flex: 1;
		padding: 10px;
		border-radius: 6px;
		font-size: 14px;
	}

	.cancel-btn {
		background: #f5f5f5;
		color: #666;
		margin-right: 10px;
	}

	.confirm-btn {
		background: #007aff;
		color: white;
	}



	/* 验证码区域样式 */
	.verify-code-section {
		display: flex;
		margin-top: 15px;
		align-items: center;
	}

	.verify-input {
		flex: 1;
		border: 1px solid #ddd;
		padding: 8px 10px;
		border-radius: 4px;
		margin-right: 10px;
	}

	.verify-btn {
		padding: 0 12px;
		height: 32px;
		line-height: 32px;
		font-size: 12px;
		white-space: nowrap;
		background-color: #f5f5f5;
		border-radius: 4px;
		color: #666;
	}

	.verify-btn[disabled] {
		color: #999;
		background-color: #f9f9f9;
	}

	/* 信息项样式 */
	.info-item {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 12px 0;
		border-bottom: 1px solid #f5f5f5;
	}

	.info-label {
		font-size: 16px;
		color: #333;
	}

	.info-value {
		display: flex;
		align-items: center;
		color: #666;
	}

	.uni-icons {
		margin-left: 8px;
	}

	/* 编辑对话框样式 */
	.edit-dialog {
		background: #fff;
		width: 80vw;
		padding: 20px;
		border-radius: 12px;
	}

	.dialog-title {
		font-size: 18px;
		font-weight: bold;
		display: block;
		text-align: center;
		margin-bottom: 20px;
	}

	.dialog-input {
		height: 2rem;
		border: 1px solid #ddd;
		padding-left: 10px;
		border-radius: 6px;
		width: 100%;
		box-sizing: border-box;
		margin-bottom: 15px;
	}

	.dialog-buttons {
		display: flex;
		justify-content: space-between;
		margin-top: 20px;
	}

	.dialog-button {
		flex: 1;
		padding: 10px;
		border-radius: 6px;
		font-size: 14px;
	}

	.dialog-button.cancel {
		background: #f5f5f5;
		color: #666;
		margin-right: 10px;
	}

	.dialog-button.confirm {
		background: #007aff;
		color: white;
	}

	/* 验证码部分样式保持不变... */
</style>