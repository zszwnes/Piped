<template>
    <h1 v-t="'titles.register'" class="my-4 text-center font-bold" />
    <hr />
    <div class="flex justify-center text-center">
        <form class="items-center px-3 children:pb-3">
            <div>
                <input
                    v-model="username"
                    class="input w-full"
                    type="text"
                    autocomplete="username"
                    :placeholder="$t('login.username')"
                    :aria-label="$t('login.username')"
                    @keyup.enter="register"
                />
            </div>
            <div class="flex justify-center">
                <input
                    v-model="password"
                    class="input w-full"
                    :type="showPassword ? 'text' : 'password'"
                    autocomplete="password"
                    :placeholder="$t('login.password')"
                    :aria-label="$t('login.password')"
                    @keyup.enter="register"
                />
                <button type="button" class="btn ml-2" @click="showPassword = !showPassword">
                    <div class="i-fa6-solid:eye" />
                </button>
            </div>
            <div class="flex justify-center">
                <input
                    v-model="passwordConfirm"
                    class="input w-full"
                    :type="showConfirmPassword ? 'text' : 'password'"
                    autocomplete="password"
                    :placeholder="$t('login.password_confirm')"
                    :aria-label="$t('login.password_confirm')"
                    @keyup.enter="register"
                />
                <button type="button" class="btn ml-2" @click="showConfirmPassword = !showConfirmPassword">
                    <div class="i-fa6-solid:eye" />
                </button>
            </div>
            <div>
                <a v-t="'titles.register'" class="btn w-auto" @click="register" />
            </div>
        </form>
    </div>
    <ConfirmModal
        v-if="showUnsecureRegisterDialog"
        :message="$t('info.register_no_email_note')"
        @close="showUnsecureRegisterDialog = false"
        @confirm="
            forceUnsecureRegister = true;
            showUnsecureRegisterDialog = false;
            register();
        "
    />
</template>

<script>
import { isEmail } from "../utils/Misc.js";
import ConfirmModal from "./ConfirmModal.vue";

export default {
    components: { ConfirmModal },
    data() {
        return {
            username: null,
            password: null,
            passwordConfirm: null,
            showPassword: false,
            showConfirmPassword: false,
            showUnsecureRegisterDialog: false,
            forceUnsecureRegister: false,
        };
    },
    mounted() {
        //TODO: Add Server Side check
        if (this.getAuthToken()) {
            this.$router.push("/");
        }
    },
    activated() {
        document.title = "Register - Piped";
    },
    methods: {
        register() {
            if (!this.username || !this.password) return;
            if (this.password != this.passwordConfirm) {
                alert(this.$t("login.passwords_incorrect"));
                return;
            }
            if (isEmail(this.username) && !this.forceUnsecureRegister) {
                this.showUnsecureRegisterDialog = true;
                return;
            }
            this.fetchJson(this.authApiUrl() + "/register", null, {
                method: "POST",
                body: JSON.stringify({
                    username: this.username,
                    password: this.password,
                }),
            }).then(resp => {
                if (resp.token) {
                    this.setPreference("authToken" + this.hashCode(this.authApiUrl()), resp.token);
                    window.location = "/"; // done to bypass cache
                } else alert(resp.error);
            });
        },
    },
};
</script>
