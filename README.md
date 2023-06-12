# NguyenGiap
from kivy.app import App
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.label import Label
from kivy.uix.button import Button
from kivy.uix.textinput import TextInput
from kivy.uix.image import Image
from kivy.uix.screenmanager import ScreenManager, Screen


class Product:
    def __init__(self, name, price, image):
        self.name = name
        self.price = price
        self.image = image


class Cart:
    def __init__(self):
        self.items = []

    def add_item(self, product):
        self.items.append(product)

    def remove_item(self, product):
        self.items.remove(product)

    def get_cart_details(self):
        cart_details = []
        for item in self.items:
            cart_details.append((item.name, item.price))
        return cart_details

    def calculate_total(self):
        total = 0
        for item in self.items:
            total += item.price
        return total

    def clear_cart(self):
        self.items = []


class LoginScreen(BoxLayout):
    def __init__(self, on_login, on_register, **kwargs):
        super().__init__(**kwargs)
        self.orientation = "vertical"
        self.spacing = 20
        self.padding = 40

        self.add_widget(Label(text="Đăng nhập", font_size=24))

        self.username_input = TextInput(hint_text="Tên đăng nhập")
        self.add_widget(self.username_input)

        self.password_input = TextInput(hint_text="Mật khẩu", password=True)
        self.add_widget(self.password_input)

        self.login_button = Button(text="Đăng nhập", size_hint=(None, None), size=(200, 40))
        self.login_button.bind(on_press=lambda instance: on_login(self.username_input.text))
        self.add_widget(self.login_button)

        self.signup_button = Button(text="Đăng ký", size_hint=(None, None), size=(200, 40))
        self.signup_button.bind(on_press=lambda instance: on_register(self.username_input.text))
        self.add_widget(self.signup_button)


class CanteenScreen(BoxLayout):
    def __init__(self, on_add_to_cart, on_remove_from_cart, on_checkout, **kwargs):
        super().__init__(**kwargs)
        self.orientation = "vertical"
        self.spacing = 20
        self.padding = 40

        self.products = [
            Product("Bim Bim ", 10000, "bimbim.jpg"),
            Product("Mỳ Ly", 20000, "myly.jpg"),
            Product("Nước ngọt", 30000, "nuocngot.jpg"),
            Product("Sữa Fami", 6000, "suafami.jpg"),
            Product("Thạch", 1500, "thach.jpg"),
            Product("Bánh mì ngọt", 8000, "banhmyngot.jpg"),
        ]

        self.add_widget(Label(text="Danh sách sản phẩm:", font_size=24))
        for product in self.products:
            product_layout = BoxLayout(orientation="horizontal", spacing=10)
            product_layout.add_widget(Image(source=product.image, size_hint=(0.2, 1)))
            product_layout.add_widget(Label(text=product.name, font_size=18))
            product_layout.add_widget(Label(text=str(product.price) + " VNĐ", font_size=18))
            add_to_cart_button = Button(text="Thêm vào giỏ hàng", size_hint=(None, None), size=(200, 40))
            add_to_cart_button.bind(on_press=lambda instance, name=product.name: on_add_to_cart(name))
            product_layout.add_widget(add_to_cart_button)
            self.add_widget(product_layout)

        self.checkout_button = Button(text="Thanh toán", size_hint=(None, None), size=(200, 40))
        self.checkout_button.bind(on_press=lambda instance: on_checkout())
        self.add_widget(self.checkout_button)


class CheckoutScreen(BoxLayout):
    def __init__(self, on_payment_complete, **kwargs):
        super().__init__(**kwargs)
        self.orientation = "vertical"
        self.spacing = 20
        self.padding = 40

        self.cart_label = Label(text="Giỏ hàng:", font_size=24)
        self.add_widget(self.cart_label)

        self.cart_items_label = Label(font_size=18)
        self.add_widget(self.cart_items_label)

        self.total_label = Label(text="Tổng tiền:", font_size=18)
        self.add_widget(self.total_label)

        self.name_input = TextInput(hint_text="Tên khách hàng", size_hint=(None, None), size=(300, 40))
        self.add_widget(self.name_input)

        self.address_input = TextInput(hint_text="Địa chỉ", size_hint=(None, None), size=(300, 40))
        self.add_widget(self.address_input)

        self.phone_input = TextInput(hint_text="Số điện thoại", size_hint=(None, None), size=(300, 40))
        self.add_widget(self.phone_input)

        self.payment_button = Button(text="Thanh toán", size_hint=(None, None), size=(200, 40))
        self.payment_button.bind(on_press=self.payment)
        self.add_widget(self.payment_button)

        self.on_payment_complete = on_payment_complete

    def update_cart_details(self, cart_details, total):
        self.cart_label.text = "Giỏ hàng:"
        self.cart_items_label.text = ""
        for item in cart_details:
            self.cart_items_label.text += "- " + item[0] + " - " + str(item[1]) + "\n"
        self.total_label.text = "Tổng tiền: " + str(total)

    def payment(self, instance):
        customer_info = {
            "name": self.name_input.text,
            "address": self.address_input.text,
            "phone": self.phone_input.text
        }
        self.on_payment_complete(customer_info)

class Order:
    def __init__(self, customer, cart):
        self.customer = customer
        self.cart = cart

    def send_order(self):
        # Gửi thông tin đơn hàng đến admin
        print("Thông tin đơn hàng:")
        print("Khách hàng:", self.customer.name)
        print("Địa chỉ:", self.customer.address)
        print("Số điện thoại:", self.customer.phone)
        print("Sản phẩm đã đặt mua:")
        for item in self.cart.items:
            print(item.name, "-", item.price)
        print("Tổng tiền:", self.cart.calculate_total())


class Customer:
    def __init__(self, name, address, phone):
        self.name = name
        self.address = address
        self.phone = phone
class CanteenApp(App):
    def build(self):
        self.icon = ("icon.png")
        self.cart = Cart()
        self.sm = ScreenManager()

        self.login_screen = LoginScreen(on_login=self.login, on_register=self.register)
        screen = Screen(name="login")
        screen.add_widget(self.login_screen)
        self.sm.add_widget(screen)

        self.canteen_screen = CanteenScreen(
            on_add_to_cart=self.add_to_cart,
            on_remove_from_cart=self.remove_from_cart,
            on_checkout=self.checkout
        )
        screen = Screen(name="canteen")
        screen.add_widget(self.canteen_screen)
        self.sm.add_widget(screen)

        self.checkout_screen = CheckoutScreen(on_payment_complete=self.payment_complete)
        screen = Screen(name="checkout")
        screen.add_widget(self.checkout_screen)
        self.sm.add_widget(screen)

        return self.sm

    def login(self, username):
        # Kiểm tra thông tin đăng nhập của admin
        if username == "admin":
            self.sm.current = "canteen"

    def register(self, username):
        # Đăng ký tài khoản mới
        # (chức năng này có thể được thêm vào trong phiên bản cải tiến)
        pass

    def add_to_cart(self, product_name):
        # Tìm sản phẩm trong danh sách sản phẩm và thêm vào giỏ hàng
        for product in self.canteen_screen.products:
            if product.name == product_name:
                self.cart.add_item(product)
                break

    def remove_from_cart(self, product_name):
        # Tìm sản phẩm trong giỏ hàng và xóa khỏi giỏ hàng
        for product in self.cart.items:
            if product.name == product_name:
                self.cart.remove_item(product)
                break

    def checkout(self):
        self.sm.current = "checkout"
        self.checkout_screen.update_cart_details(self.cart.get_cart_details(), self.cart.calculate_total())

    def payment_complete(self, customer_info):
        # Tạo đối tượng khách hàng từ thông tin được nhập vào
        customer = Customer(customer_info["name"], customer_info["address"], customer_info["phone"])
        order = Order(customer, self.cart)
        order.send_order()
        self.cart.clear_cart()
        self.sm.current = "canteen"


if __name__ == '__main__':
    CanteenApp().run()
