<!--
Copyright 2020 DigitalOcean

This code is licensed under the MIT License.
You may obtain a copy of the License at
https://github.com/digitalocean/nginxconfig.io/blob/master/LICENSE or https://mit-license.org/

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and / or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions :

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
-->

<template>
    <div>
        <div class="field is-horizontal">
            <div class="field-label">
                <label class="label">{{ i18n.common.https }}</label>
            </div>
            <div class="field-body">
                <div class="field">
                    <div :class="`control${httpsChanged ? ' is-changed' : ''}`">
                        <div class="checkbox">
                            <PrettyCheck v-model="https" class="p-default p-curve p-fill p-icon">
                                <i slot="extra" class="icon fas fa-check"></i>
                                {{ i18n.templates.domainSections.https.enableEncryptedSslConnection }}
                            </PrettyCheck>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <div v-if="http2Enabled" class="field is-horizontal">
            <div class="field-label">
                <label class="label">{{ i18n.templates.domainSections.https.http2 }}</label>
            </div>
            <div class="field-body">
                <div class="field">
                    <div :class="`control${http2Changed ? ' is-changed' : ''}`">
                        <div class="checkbox">
                            <PrettyCheck v-model="http2" class="p-default p-curve p-fill p-icon">
                                <i slot="extra" class="icon fas fa-check"></i>
                                {{ i18n.templates.domainSections.https.enableHttp2Connections }}
                            </PrettyCheck>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <div v-if="forceHttpsEnabled" class="field is-horizontal">
            <div class="field-label">
                <label class="label">{{ i18n.templates.domainSections.https.forceHttps }}</label>
            </div>
            <div class="field-body">
                <div class="field">
                    <div :class="`control${forceHttpsChanged ? ' is-changed' : ''}`">
                        <div class="checkbox">
                            <PrettyCheck v-model="forceHttps" class="p-default p-curve p-fill p-icon">
                                <i slot="extra" class="icon fas fa-check"></i>
                                (http://{{ $parent.$props.data.server.domain.computed }}
                                <i class="fas fa-long-arrow-alt-right"></i>
                                https://{{ $parent.$props.data.server.domain.computed }})
                            </PrettyCheck>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <div v-if="hstsEnabled" class="field is-horizontal is-aligned-top">
            <div class="field-label">
                <label class="label">{{ i18n.templates.domainSections.https.hsts }}</label>
            </div>
            <div class="field-body">
                <div class="field">
                    <div :class="`control${hstsChanged ? ' is-changed' : ''}`">
                        <div class="checkbox">
                            <PrettyCheck v-model="hsts" class="p-default p-curve p-fill p-icon">
                                <i slot="extra" class="icon fas fa-check"></i>
                                {{ i18n.templates.domainSections.https.enableStrictTransportSecurity }}
                            </PrettyCheck>
                        </div>
                    </div>

                    <div v-if="hstsSubdomainsEnabled" :class="`control${hstsSubdomainsChanged ? ' is-changed' : ''}`">
                        <div class="checkbox">
                            <PrettyCheck v-model="hstsSubdomains" class="p-default p-curve p-fill p-icon">
                                <i slot="extra" class="icon fas fa-check"></i>
                                includeSubDomains
                            </PrettyCheck>
                        </div>
                    </div>

                    <div v-if="hstsPreloadEnabled" :class="`control${hstsPreloadChanged ? ' is-changed' : ''}`">
                        <div class="checkbox">
                            <PrettyCheck v-model="hstsPreload" class="p-default p-curve p-fill p-icon">
                                <i slot="extra" class="icon fas fa-check"></i>
                                preload
                            </PrettyCheck>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <div v-if="certTypeEnabled" class="field is-horizontal is-aligned-top">
            <div class="field-label">
                <label class="label">{{ i18n.templates.domainSections.https.certificationType }}</label>
            </div>
            <div class="field-body">
                <div class="field">
                    <div v-for="(name, value) in $props.data.certType.options"
                         :class="`control${certTypeChanged && value === certType ? ' is-changed' : ''}`"
                    >
                        <div class="radio">
                            <PrettyRadio v-model="certType" :value="value" class="p-default p-round p-fill p-icon">
                                <i slot="extra" class="icon fas fa-check"></i>
                                {{ name }}
                            </PrettyRadio>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <div v-if="letsEncryptEmailEnabled" class="field is-horizontal">
            <div class="field-label">
                <label class="label">{{ i18n.templates.domainSections.https.letsEncryptEmail }}</label>
            </div>
            <div class="field-body">
                <div class="field">
                    <div :class="`control${letsEncryptEmailChanged ? ' is-changed' : ''}`">
                        <input v-model="letsEncryptEmail"
                               class="input"
                               type="text"
                               :placeholder="$props.data.letsEncryptEmail.computed"
                        />
                    </div>
                </div>
            </div>
        </div>

        <div v-if="sslCertificateEnabled" class="field is-horizontal">
            <div class="field-label">
                <label class="label">ssl_certificate</label>
            </div>
            <div class="field-body">
                <div class="field">
                    <div :class="`control${sslCertificateChanged ? ' is-changed' : ''}`">
                        <input v-model="sslCertificate"
                               class="input"
                               type="text"
                               :placeholder="`${$parent.$parent.$data.global.nginx.nginxConfigDirectory.computed}/ssl/${$parent.$props.data.server.domain.computed}.crt`"
                        />
                    </div>
                </div>
            </div>
        </div>

        <div v-if="sslCertificateKeyEnabled" class="field is-horizontal">
            <div class="field-label">
                <label class="label">ssl_certificate_key</label>
            </div>
            <div class="field-body">
                <div class="field">
                    <div :class="`control${sslCertificateKeyChanged ? ' is-changed' : ''}`">
                        <input v-model="sslCertificateKey"
                               class="input"
                               type="text"
                               :placeholder="`${$parent.$parent.$data.global.nginx.nginxConfigDirectory.computed}/ssl/${$parent.$props.data.server.domain.computed}.key`"
                        />
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    import PrettyCheck from 'pretty-checkbox-vue/check';
    import PrettyRadio from 'pretty-checkbox-vue/radio';
    import i18n from '../../i18n';
    import delegatedFromDefaults from '../../util/delegated_from_defaults';
    import computedFromDefaults from '../../util/computed_from_defaults';

    const defaults = {
        https: {
            default: true,
            enabled: true,
        },
        http2: {
            default: true,
            enabled: true,
        },
        forceHttps: {
            default: true,
            enabled: true,
        },
        hsts: {
            default: true,
            enabled: true,
        },
        hstsSubdomains: {
            default: true,
            enabled: true,
        },
        hstsPreload: {
            default: true,
            enabled: true,
        },
        certType: {
            default: 'letsEncrypt',
            options: {
                letsEncrypt: i18n.common.letsEncrypt,
                custom: i18n.templates.domainSections.https.customCertificate,
            },
            enabled: true,
        },
        letsEncryptEmail: {
            default: '',
            computed: 'info@example.com', // No default value, but a default computed
            enabled: true,
        },
        sslCertificate: {
            default: '',
            enabled: false,
        },
        sslCertificateKey: {
            default: '',
            enabled: false,
        },
    };

    export default {
        name: 'DomainHTTPS',                                // Component name
        display: i18n.common.https,                         // Display name for tab
        key: 'https',                                       // Key for data in parent
        delegated: delegatedFromDefaults(defaults),         // Data the parent will present here
        components: {
            PrettyCheck,
            PrettyRadio,
        },
        props: {
            data: Object,                                   // Data delegated back to us from parent
        },
        data () {
            return {
                i18n,
            };
        },
        computed: computedFromDefaults(defaults, 'https'),  // Getters & setters for the delegated data
        watch: {
            // Disable everything if https is disabled
            '$props.data.https': {
                handler(data) {
                    const state = data.computed;
                    if (state) {
                        this.$props.data.http2.enabled = true;
                        this.$props.data.http2.computed = this.$props.data.http2.value;
                        this.$props.data.forceHttps.enabled = true;
                        this.$props.data.forceHttps.computed = this.$props.data.forceHttps.value;
                        this.$props.data.hsts.enabled = true;
                        this.$props.data.hsts.computed = this.$props.data.hsts.value;
                        this.$props.data.certType.enabled = true;
                        this.$props.data.certType.computed = this.$props.data.certType.value;
                    } else {
                        this.$props.data.http2.enabled = false;
                        this.$props.data.http2.computed = false;
                        this.$props.data.forceHttps.enabled = false;
                        this.$props.data.forceHttps.computed = false;
                        this.$props.data.hsts.enabled = false;
                        this.$props.data.hsts.computed = false;
                        this.$props.data.certType.enabled = false;
                        this.$props.data.certType.computed = '';
                    }
                },
                deep: true,
            },
            // Disable hsts options if hsts is disabled
            '$props.data': {
                handler() {
                    // hstsSubdomains
                    if (this.$props.data.hsts.computed) {
                        this.$props.data.hstsSubdomains.enabled = true;
                        this.$props.data.hstsSubdomains.computed = this.$props.data.hstsSubdomains.value;
                    } else {
                        this.$props.data.hstsSubdomains.enabled = false;
                        this.$props.data.hstsSubdomains.computed = false;
                    }

                    // hstsPreload
                    if (this.$props.data.hsts.computed && this.$props.data.hstsSubdomains.computed) {
                        this.$props.data.hstsPreload.enabled = true;
                        this.$props.data.hstsPreload.computed = this.$props.data.hstsPreload.value;
                    } else {
                        this.$props.data.hstsPreload.enabled = false;
                        this.$props.data.hstsPreload.computed = false;
                    }
                },
                deep: true,
            },
            // Toggle form elms based on cert selection
            '$props.data.certType': {
                handler(data) {
                    // This might cause recursion, but seems not to
                    // Hide all if disabled
                    if (!data.enabled) {
                        this.$props.data.letsEncryptEmail.enabled = false;
                        this.$props.data.letsEncryptEmail.computed = '';
                        this.$props.data.sslCertificate.enabled = false;
                        this.$props.data.sslCertificate.computed = '';
                        this.$props.data.sslCertificateKey.enabled = false;
                        this.$props.data.sslCertificateKey.computed = '';
                    } else {
                        // First, check its valid
                        if (!Object.keys(data.options).includes(data.computed)) data.computed = data.default;

                        // Show the correct fields
                        if (data.computed === 'letsEncrypt') {
                            this.$props.data.letsEncryptEmail.enabled = true;
                            this.$props.data.letsEncryptEmail.computed = this.$props.data.letsEncryptEmail.value;

                            this.$props.data.sslCertificate.enabled = false;
                            this.$props.data.sslCertificate.computed = '';
                            this.$props.data.sslCertificateKey.enabled = false;
                            this.$props.data.sslCertificateKey.computed = '';
                        } else {
                            this.$props.data.sslCertificate.enabled = true;
                            this.$props.data.sslCertificate.computed = this.$props.data.sslCertificate.value;
                            this.$props.data.sslCertificateKey.enabled = true;
                            this.$props.data.sslCertificateKey.computed = this.$props.data.sslCertificateKey.value;

                            this.$props.data.letsEncryptEmail.enabled = false;
                            this.$props.data.letsEncryptEmail.computed = '';
                        }
                    }
                },
                deep: true,
            },
            // Ensure there is a default email for Let's Encrypt
            '$props.data.letsEncryptEmail': {
                handler(data) {
                    if (!data.computed.trim()) {
                        data.computed = `info@${this.$parent.$props.data.server.domain.computed}`;
                    }
                },
                deep: true,
            },
            '$parent.$props.data.server.domain': {
                handler(data) {
                    if (!this.$props.data.letsEncryptEmail.value.trim()) {
                        this.$props.data.letsEncryptEmail.computed = `info@${data.computed}`;
                    }
                },
                deep: true,
            },
        },
    };
</script>
