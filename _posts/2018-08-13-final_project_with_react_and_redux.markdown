---
layout: post
title:      "Final Project with React and Redux"
date:       2018-08-13 22:40:32 -0400
permalink:  final_project_with_react_and_redux
---


My app is to create a order management app, which was a bit large scale. 
Main goal was to get used to with handling the data with React  and Redux, and used Semantic-UI React for different views.
On the backend, I set up Rails API with postgreSQL with active serializer and Devise.

1. Authentication
I used JsonWebToken for authentication, and the hard part was to combine the JsonWebToken with React-Router logic, and handling this in the backend with Devise.
Researched and tested many ways and I decided to wrap the protected routes with one file that checks the authentication. 
All protected routes are inside `AuthenticatedRoutes`. In `AuthenticatedRoutes`, it has multiple API calls to check authentication and get the data needed upfront in `componentDidMount()`.
In the API backend, I checked several ways to working with JsonWebToken with Devise, but ended up with customizing the Devise routes to handle Authentication through JsonWebToken.

```
class App extends Component {
  render() {
    return (
      <BrowserRouter>
        <Switch>
          <Route exact path="/signup" component={SignUp} />
          <Route exact path="/signin" component={SignIn} />
          <AuthenticatedRoutes>
            <Header />
            <Route exact path="/" component={Dashboard} />
            <Route exact path="/dashboard" component={Dashboard} />
            <Route exact path="/demand" component={Demand} />
              <Route path="/customers" component={Customers} />
              <Route path="/sales" component={SalesOrders} />
            <Route exact path="/supply" component={Supply} />
              <Route path="/suppliers" component={Suppliers} />
              <Route path="/purchases" component={PurchaseOrders} />
            <Route exact path="/product" component={Product} />
              <Route path="/products" component={Products} />
            <Route exact path="/logistics" component={Logistics} />
            <Route exact path="/warehouse" component={Warehouse} />
            <Route exact path="/setting" component={Setting} />
              <Route path="/account-addresses" component={AccountAddresses} />
              <Route path="/account-contacts" component={AccountContacts} />
              <Route path="/payment-terms" component={PaymentTerms} />
              <Route path="/product-brands" component={ProductBrands} />
              <Route path="/product-categories" component={ProductCategories} />
              <Route path="/warehouses" component={Warehouses} />

            <Route path="/accounts" component={AccountsPage} />
            <Footer />
          </AuthenticatedRoutes>
        </Switch>
      </BrowserRouter>
    );
  }
}

function mapStateToProps({ user }) {
  return {
    currentAccount: user.currentAccount,
    currentUser: user.currentUser
  };
}

function mapDispatchToProps(dispatch) {
  return {
    actions: bindActionCreators(actions, dispatch)
  };
}

export default connect(
  mapStateToProps,
  mapDispatchToProps
)(App);
```


2. Data Loading with Redux
I wanted to display the loading bar, and I added loading status with multiple steps within one action.
For example, I set up action type: "GET_CURRENT_ACCOUNT_BEGIN" and "GET_CURRENT_ACCOUNT_SUCCESS".
In reducer, those action types updates `currentAccountLoading` status. 
When it begins, `currentAccountLoading`  is true, in React redering, there is condition for this to render `<Loading />` view. Then, when `currentAccountLoading` becomes false by the execution of "GET_CURRENT_ACCOUNT_SUCCESS", it renders page with actual data.

```
userActions.js -
export const getCurrentAccount = data =>  async dispatch => {
  try {
    dispatch({ type: "GET_CURRENT_ACCOUNT_BEGIN" });
    const res = await axios({
      method: "GET",
      url: "/api/v1/current_account",
      headers: {
        Authorization: `Bearer ${localStorage.getItem("token")}`,
        "Content-Type": "application/json"
      },
      data: data
    })
    
    dispatch({ type: "GET_CURRENT_ACCOUNT_SUCCESS", payload: res.data })
  } catch(err) {
    dispatch({ type: "GET_CURRENT_ACCOUNT_FAILURE", payload: err })
  }
};

Header.js -
class Header extends Component {
  constructor(props) {
    super(props);

    this.state = {
      activeItem: window.location.pathname,
      currentAccount: props.currentAccount
    };
  }

  render() {
    const { accounts, currentAccount, currentAccountLoading } = this.props;

    const accountsDropdownList = accounts.map(account => {
      return (
        <Dropdown.Item
          data-id={account.id}
          key={account.id}
          text={account.name}
          value={account.id}
          onClick={this.handleAccountClick}
        />
      );
    });

    if (currentAccountLoading) {
      return <Loading />;
    }

    return (
      <React.Fragment>
        <header>
          <Menu pointing size="small">
            <Menu.Item icon="bars" fitted="vertically" />
            <Menu.Menu>
              <Dropdown
                item
                text={currentAccount ? currentAccount.name : "my app"}
              >
                <Dropdown.Menu>
                  <Dropdown.Header>
                    <a href="/accounts">My Accounts</a>
                  </Dropdown.Header>
                  <Dropdown.Divider />
                  {!!accounts.length && accountsDropdownList}
                </Dropdown.Menu>
              </Dropdown>
            </Menu.Menu>

            <Menu.Menu position="right">
              <div className="ui right aligned category search item">
                <div className="ui transparent icon input">
                  <input
                    className="prompt"
                    type="text"
                    placeholder="Search..."
                  />
                  <i className="search link icon" />
                </div>
                <div className="results" />
              </div>

              <Dropdown item icon="user" simple>
                <Dropdown.Menu>
                  <Dropdown.Item>
                    <Icon name="dropdown" />
                    <span className="text">New</span>

                    <Dropdown.Menu>
                      <Dropdown.Item>Document</Dropdown.Item>
                      <Dropdown.Item>Image</Dropdown.Item>
                    </Dropdown.Menu>
                  </Dropdown.Item>
                  <Dropdown.Item>Open</Dropdown.Item>
                  <Dropdown.Item>Save...</Dropdown.Item>
                  <Dropdown.Item>Edit Permissions</Dropdown.Item>
                  <Dropdown.Divider />
                  <Dropdown.Header>Export</Dropdown.Header>
                  <Dropdown.Item>Share</Dropdown.Item>
                </Dropdown.Menu>
              </Dropdown>

              <Menu.Item onClick={event => this.handleSignOut(event)}>
                Sign out
              </Menu.Item>
            </Menu.Menu>
          </Menu>

          <Menu className="padding-all-sm" pointing secondary size="tiny">
            <Menu.Item
              as={Link}
              to="/dashboard"
              name="dashboard"
              active={this.state.activeItem === "/dashboard"}
              onClick={this.handleItemClick}
            />
            <Menu.Item
              as={Link}
              to="/demand"
              name="demand"
              active={this.state.activeItem === "/demand"}
              onClick={this.handleItemClick}
            />
            <Menu.Item
              as={Link}
              to="/supply"
              name="supply"
              active={this.state.activeItem === "/supply"}
              onClick={this.handleItemClick}
            />
            <Menu.Item
              as={Link}
              to="/product"
              name="product"
              active={this.state.activeItem === "/product"}
              onClick={this.handleItemClick}
            />
            <Menu.Item
              as={Link}
              to="/logistics"
              name="logistics"
              active={this.state.activeItem === "/logistics"}
              onClick={this.handleItemClick}
            />
            <Menu.Item
              as={Link}
              to="/warehouse"
              name="warehouse"
              active={this.state.activeItem === "/warehouse"}
              onClick={this.handleItemClick}
            />
            <Menu.Item
              as={Link}
              to="/setting"
              name="setting"
              active={this.state.activeItem === "/setting"}
              onClick={this.handleItemClick}
            />
          </Menu>
        </header>
      </React.Fragment>
    );
  }
}

function mapStateToProps({ accounts, user }) {
  return {
    accounts: accounts.accounts,
    currentAccount: user.currentAccount,
    currentAccountLoading: user.currentAccountLoading
  };
}

function mapDispatchToProps(dispatch) {
  return {
    actions: bindActionCreators(actions, dispatch)
  };
}

export default withRouter(
  connect(
    mapStateToProps,
    mapDispatchToProps
  )(Header)
);

```


3. Dynamic form changes
I tried to do the dynamic form changes with React.
It looks very small, but apparently there are a lot of things going on with this. 

Basically, idea is that we should add empty object into array in the state, then rendering this.
I created function to handle each adding, removing, and changing the line dynamically.

```

  handleAddOrderLine = e => {
    e.preventDefault();
    this.setState({
      ...this.state,
      purchase_order: {
        ...this.state.purchase_order,
        order_lines_attributes: this.state.purchase_order.order_lines_attributes.concat(
          {
            id: null,
            order_id: null,
            product_id: "",
            comment: "",
            quantity: 1,
            unit_price: 0,
            _destroy: false
          }
        )
      }
    });
  };

  handleRemoveOrderLine = (e, index) => {
    e.preventDefault();
    console.log("remove line");
    const stateOrderLines = this.state.purchase_order.order_lines_attributes;

    if (!!stateOrderLines[index].id) {
      const newOrderLines = stateOrderLines.map((line, stateIndex) => {
        if (stateIndex !== index) return line;
        return Object.assign({}, line, { _destroy: true });
      });

      this.setState({
        ...this.state,
        purchase_order: {
          ...this.state.purchase_order,
          order_lines_attributes: newOrderLines
        }
      });
    } else {
      this.setState({
        ...this.state,
        purchase_order: {
          ...this.state.purchase_order,
          order_lines_attributes: stateOrderLines.filter(
            (line, stateIndex) => index !== stateIndex
          )
        }
      });
    }
  };

  handleOrderLineChange = (e, index) => {
    const key =
      e.target.name ||
      (e.target.parentNode.dataset && e.target.parentNode.dataset.name) ||
      (e.target.closest(".selected") &&
        e.target.closest(".selected").dataset.name);

    const value =
      e.target.value ||
      (e.target.parentNode.dataset && e.target.parentNode.dataset.value) ||
      (e.target.closest(".selected") &&
        e.target.closest(".selected").dataset.value);

    const newOrderLines = this.state.purchase_order.order_lines_attributes.map(
      (line, stateIndex) => {
        if (stateIndex !== index) return line;
        return Object.assign({}, line, { [key]: value });
      }
    );

    this.setState({
      ...this.state,
      purchase_order: {
        ...this.state.purchase_order,
        order_lines_attributes: newOrderLines
      }
    });
  };

```


I added helpful references below.


References - 
https://www.youtube.com/watch?v=jJH69vTLcuQ
https://www.youtube.com/watch?v=Nq9RmAB9eag&t=320s
https://daveceddia.com/where-fetch-data-redux/
https://goshakkk.name/array-form-inputs/



