/**
 * User operation handlers
 */
const userOperations = {
  login: async (params) => {
    const { phoneNumber, password } = params;
    const validation = validateLoginInput(phoneNumber, password);
    if (!validation.isValid) {
      return createErrorResponse(validation.error);
    }

    const user = await queries.findUserByPhone(phoneNumber);
    if (!user) {
      return createErrorResponse(ERROR_MESSAGES.AUTH.INVALID_CREDENTIALS);
    }

    try {
      const loggedInUser = await Parse.User.logIn(
        user.get('username'),
        password
      );
      await queries.updateLastOnline(loggedInUser.id);

      return createSuccessResponse({
        user: {
          id: loggedInUser.id,
          username: loggedInUser.get('username'),
          email: loggedInUser.get('email'),
          phoneNumber: loggedInUser.get('phoneNumber'),
          language: loggedInUser.get('language'),
          lastOnline: loggedInUser.get('lastOnline'),
        },
      });
    } catch (error) {
      console.error('Login error:', error);
      return createErrorResponse(ERROR_MESSAGES.AUTH.INVALID_CREDENTIALS);
    }
  },

  register: async (params) => {
    const userData = params;
    const username = `user_${Date.now()}`;

    const existingPhone = await queries.findUserByPhone(userData.phoneNumber);
    if (existingPhone) {
      return createErrorResponse(ERROR_MESSAGES.ACCOUNT.ALREADY_EXISTS);
    }

    if (userData.email) {
      const existingEmail = await queries.findUserByEmail(userData.email);
      if (existingEmail) {
        return createErrorResponse(ERROR_MESSAGES.ACCOUNT.EMAIL_EXISTS);
      }
    }

    const user = new Parse.User();
    user.set('username', username);
    user.set('phoneNumber', userData.phoneNumber);
    user.set('email', userData.email);
    user.set('language', userData.language);
    user.set('sex', userData.sex);
    user.set('skinColor', userData.skinColor);
    user.set('deviceInfos', userData.deviceInfos);
    user.set('userState', 'unlocked');
    user.set('lastOnline', new Date());

    if (!userData.isSocialLogin && userData.password) {
      user.set('password', userData.password);
    }

    try {
      const savedUser = await user.signUp();
      return createSuccessResponse({ user: savedUser.toJSON() });
    } catch (error) {
      console.error('Registration error:', error);
      return createErrorResponse(ERROR_MESSAGES.ACCOUNT.REGISTRATION_FAILED);
    }
  },

  requestPasswordReset: async (params) => {
    const { email } = params;
    const validation = validatePasswordResetInput(email);
    if (!validation.isValid) {
      return createErrorResponse(validation.error);
    }

    const user = await queries.findUserByEmail(email);
    try {
      if (user) {
        await Parse.User.requestPasswordReset(email);
      }
      return createSuccessResponse({
        message:
          'Si un compte existe avec cet email, vous recevrez un email avec les instructions de réinitialisation.',
      });
    } catch (error) {
      console.error('Password reset error:', error);
      return createErrorResponse(ERROR_MESSAGES.SERVER.GENERAL_ERROR);
    }
  },

  updateDevice: async (params) => {
    const { userId, deviceInfos } = params;
    try {
      const updatedUser = await queries.updateUserDevice(userId, deviceInfos);
      return createSuccessResponse({ user: updatedUser.toJSON() });
    } catch (error) {
      console.error('Update device error:', error);
      return createErrorResponse(ERROR_MESSAGES.SERVER.GENERAL_ERROR);
    }
  },

  updateLastOnline: async (params) => {
    const { userId } = params;
    try {
      const updatedUser = await queries.updateLastOnline(userId);
      return createSuccessResponse({ user: updatedUser.toJSON() });
    } catch (error) {
      console.error('Update last online error:', error);
      return createErrorResponse(ERROR_MESSAGES.SERVER.GENERAL_ERROR);
    }
  },
};

module.exports = userOperations;
