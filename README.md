# Automated-Tests-Saucedemo
Tests — Saucedemo
/test
  /pageobjects
    LoginPage.js
    InventoryPage.js
    CartPage.js
    CheckoutPage.js
  /specs
    login.spec.js
    cart.spec.js
    sorting.spec.js
    checkout.spec.js
    logout.spec.js
class LoginPage {
    get inputUsername() { return ('#user-name'); }
    get inputPassword() { return ('#password'); }
    get btnLogin() { return ('#login-button'); }
    get errorMessage() { return ('h3[data-test="error"]'); }

    async open() {
        await browser.url('https://www.saucedemo.com/');
    }

    async login(username, password) {
        await this.inputUsername.setValue(username);
        await this.inputPassword.setValue(password);
        await this.btnLogin.click();
    }
module.exports = new LoginPage();
class InventoryPage 
    get cartBadge() { return ('.shopping_cart_badge'); }
    get cartLink() { return ('.shopping_cart_link'); }
    get sortDropdown() { return ('.product_sort_container'); }

    async addItem(itemSelector) {
        await (itemSelector).click();
    }

    async openCart() {
        await this.cartLink.click();
    }

    async sortBy(option) {
        await this.sortDropdown.selectByVisibleText(option);
    }
module.exports = new InventoryPage();
class CartPage {
    get cartItems() { return ('.cart_item'); }
    get checkoutBtn() { return ('#checkout'); }

    async getCartItemsCount() {
        return await this.cartItems.length;
    }

    async checkout() {
        await this.checkoutBtn.click();
    }
}
module.exports = new CartPage();
class CheckoutPage {
    get firstName() { return ('#first-name'); }
    get lastName() { return ('#last-name'); }
    get postalCode() { return ('#postal-code'); }
    get continueBtn() { return ('#continue'); }
    get finishBtn() { return ('#finish'); }
    get errorMessage() { return ('h3[data-test="error"]'); }
    get successMessage() { return ('.complete-header'); }

    async fillForm(first, last, code) {
        await this.firstName.setValue(first);
        await this.lastName.setValue(last);
        await this.postalCode.setValue(code);
        await this.continueBtn.click();
    }

    async finishCheckout() {
        await this.finishBtn.click();
    }
}
module.exports = new CheckoutPage();
const LoginPage = require('../pageobjects/LoginPage');

describe('Login tests', () => {
    it('TC01: Valid login', async () => {
        await LoginPage.open();
        await LoginPage.login('standard_user', 'secret_sauce');
        await expect(browser).toHaveUrlContaining('/inventory.html');
    });

    it('TC02: Invalid login', async () => {
        await LoginPage.open();
        await LoginPage.login('standard_user', 'wrong_password');
        await expect(LoginPage.errorMessage).toHaveTextContaining('Username and password do not match');
    });

    it('TC03: Locked out user', async () => {
        await LoginPage.open();
        await LoginPage.login('locked_out_user', 'secret_sauce');
        await expect(LoginPage.errorMessage).toHaveTextContaining('Sorry, this user has been locked out');
    });
});
const LoginPage = require('../pageobjects/LoginPage');
const InventoryPage = require('../pageobjects/InventoryPage');
const CartPage = require('../pageobjects/CartPage');

describe('Cart tests', () => {
    beforeEach(async () => {
        await LoginPage.open();
        await LoginPage.login('standard_user', 'secret_sauce');
    });

    it('TC04: Add item to cart', async () => {
        await InventoryPage.addItem('button[data-test="add-to-cart-sauce-labs-backpack"]');
        await expect(InventoryPage.cartBadge).toHaveText('1');
    });

    it('TC05: Remove item from cart', async () => {
        await InventoryPage.addItem('button[data-test="add-to-cart-sauce-labs-backpack"]');
        await InventoryPage.openCart();
        await ('button[data-test="remove-sauce-labs-backpack"]').click();
        await expect(InventoryPage.cartBadge).not.toExist();
    });
});
const LoginPage = require('../pageobjects/LoginPage');
const InventoryPage = require('../pageobjects/InventoryPage');

describe('Sorting tests', () => {
    beforeEach(async () => {
        await LoginPage.open();
        await LoginPage.login('standard_user', 'secret_sauce');
    });

    it('TC06: Sort by Price low→high', async () => {
        await InventoryPage.sortBy('Price (low to high)');
        // тут можна додати перевірку порядку цін
    });

    it('TC07: Sort by Price high→low', async () => {
        await InventoryPage.sortBy('Price (high to low)');
    });

    it('TC08: Sort by Name A→Z', async () => {
        await InventoryPage.sortBy('Name (A to Z)');
    });
});
const LoginPage = require('../pageobjects/LoginPage');
const InventoryPage = require('../pageobjects/InventoryPage');
const CartPage = require('../pageobjects/CartPage');
const CheckoutPage = require('../pageobjects/CheckoutPage');

describe('Checkout tests', () => {
    beforeEach(async () => {
        await LoginPage.open();
        await LoginPage.login('standard_user', 'secret_sauce');
        await InventoryPage.addItem('button[data-test="add-to-cart-sauce-labs-backpack"]');
        await InventoryPage.openCart();
        await CartPage.checkout();
    });

    it('TC09: Successful checkout', async () => {
        await CheckoutPage.fillForm('Tetiana', 'QA', '12345');
        await CheckoutPage.finishCheckout();
        await expect(CheckoutPage.successMessage).toHaveTextContaining('THANK YOU FOR YOUR ORDER');
    });

    it('TC10: Validation error in checkout', async () => {
        await CheckoutPage.fillForm('', '', '');
        await expect(CheckoutPage.errorMessage).toHaveTextContaining('Error: First Name is required');
    });
});
const LoginPage = require('../pageobjects/LoginPage');

describe('Logout test', () => {
    it('TC11: Logout', async () => {
        await LoginPage.open();
        await LoginPage.login('standard_user', 'secret_sauce');
        await ('#react-burger-menu-btn').click();
        await ('#logout_sidebar_link').click();
        await expect(browser).toHaveUrl('https://www.saucedemo.com/');
    });
});
npx wdio run wdio.conf.js
